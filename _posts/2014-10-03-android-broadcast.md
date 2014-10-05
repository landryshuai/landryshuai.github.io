---
layout: post
title: "Android Broadcast"
description: ""
keywords: ""
category: [work]
tags: [Android]
---
# Android Broadcast总结

在ActivityManagerService.java里面
`private final int broadcastIntentLocked(ProcessRecord callerApp,
String callerPackage, Intent intent, String resolvedType,
IIntentReceiver resultTo, int resultCode, String resultData,
Bundle map, String requiredPermission,
boolean ordered, boolean sticky, int callingPid, int callingUid)`
这个函数，有几个域比较重要的：  
`mParallelBroadcasts` 记录并行的广播列表。  
`mParallelBroadcasts` 记录order的广播列表。  
`mStickyBroadcasts` 记录sticky的广播列表。  
{% highlight java %}
// The first sticky in the list is returned directly back to
// the client.
Intent sticky = allSticky != null ? (Intent)allSticky.get(0) : null;
{% endhighlight %}
StickyBroadcasts的作用在与，当注册的时候，可以获取最后一自己注册的广播。  

下面一段代码是处理琐碎事情，如删除掉已经删掉的packages、发广播的权限处理等 
{% highlight java %}
intent = new Intent(intent);
// By default broadcasts do not go to stopped apps.
intent.addFlags(Intent.FLAG_EXCLUDE_STOPPED_PACKAGES);
if (DEBUG_BROADCAST_LIGHT) Slog.v(
TAG, (sticky ? "Broadcast sticky: ": "Broadcast: ") + intent
+ " ordered=" + ordered);
if ((resultTo != null) && !ordered) {
Slog.w(TAG, "Broadcast " + intent + " not ordered but result callback requested!");
}

// Handle special intents: if this broadcast is from the package
// manager about a package being removed, we need to remove all of
// its activities from the history stack.
final boolean uidRemoved = Intent.ACTION_UID_REMOVED.equals(
intent.getAction());
if (Intent.ACTION_PACKAGE_REMOVED.equals(intent.getAction())
|| Intent.ACTION_PACKAGE_CHANGED.equals(intent.getAction())
|| Intent.ACTION_EXTERNAL_APPLICATIONS_UNAVAILABLE.equals(intent.getAction())
|| uidRemoved) {
if (checkComponentPermission(
android.Manifest.permission.BROADCAST_PACKAGE_REMOVED,
callingPid, callingUid, -1, true)
== PackageManager.PERMISSION_GRANTED) {
if (uidRemoved) {
final Bundle intentExtras = intent.getExtras();
final int uid = intentExtras != null
? intentExtras.getInt(Intent.EXTRA_UID) : -1;
if (uid >= 0) {
BatteryStatsImpl bs = mBatteryStatsService.getActiveStatistics();
synchronized (bs) {
bs.removeUidStatsLocked(uid);
}
}
} else {
// If resources are unvailble just force stop all
// those packages and flush the attribute cache as well.
if (Intent.ACTION_EXTERNAL_APPLICATIONS_UNAVAILABLE.equals(intent.getAction())) {
String list[] = intent.getStringArrayExtra(Intent.EXTRA_CHANGED_PACKAGE_LIST);
if (list != null && (list.length > 0)) {
for (String pkg : list) {
forceStopPackageLocked(pkg, -1, false, true, true, false);
}
sendPackageBroadcastLocked(
IApplicationThread.EXTERNAL_STORAGE_UNAVAILABLE, list);
}
} else {
Uri data = intent.getData();
String ssp;
if (data != null && (ssp=data.getSchemeSpecificPart()) != null) {
if (!intent.getBooleanExtra(Intent.EXTRA_DONT_KILL_APP, false)) {
forceStopPackageLocked(ssp,
intent.getIntExtra(Intent.EXTRA_UID, -1), false, true, true, false);
}
if (Intent.ACTION_PACKAGE_REMOVED.equals(intent.getAction())) {
sendPackageBroadcastLocked(IApplicationThread.PACKAGE_REMOVED,
new String[] {ssp});
}
}
}
}
} else {
String msg = "Permission Denial: " + intent.getAction()
+ " broadcast from " + callerPackage + " (pid=" + callingPid
+ ", uid=" + callingUid + ")"
+ " requires "
+ android.Manifest.permission.BROADCAST_PACKAGE_REMOVED;
Slog.w(TAG, msg);
throw new SecurityException(msg);
}
// Special case for adding a package: by default turn on compatibility
// mode.
} else if (Intent.ACTION_PACKAGE_ADDED.equals(intent.getAction())) {
Uri data = intent.getData();
String ssp;
if (data != null && (ssp=data.getSchemeSpecificPart()) != null) {
mCompatModePackages.handlePackageAddedLocked(ssp,
intent.getBooleanExtra(Intent.EXTRA_REPLACING, false));
}
}
/*
* If this is the time zone changed action, queue up a message that will reset the timezone
* of all currently running processes. This message will get queued up before the broadcast
* happens.
*/
if (intent.ACTION_TIMEZONE_CHANGED.equals(intent.getAction())) {
mHandler.sendEmptyMessage(UPDATE_TIME_ZONE);
}
if (intent.ACTION_CLEAR_DNS_CACHE.equals(intent.getAction())) {
mHandler.sendEmptyMessage(CLEAR_DNS_CACHE);
}
if (Proxy.PROXY_CHANGE_ACTION.equals(intent.getAction())) {
ProxyProperties proxy = intent.getParcelableExtra("proxy");
mHandler.sendMessage(mHandler.obtainMessage(UPDATE_HTTP_PROXY, proxy));
}
/*
* Prevent non-system code (defined here to be non-persistent
* processes) from sending protected broadcasts.
*/
if (callingUid == Process.SYSTEM_UID || callingUid == Process.PHONE_UID
|| callingUid == Process.SHELL_UID || callingUid == 0) {
// Always okay.
} else if (callerApp == null || !callerApp.persistent) {
try {
if (AppGlobals.getPackageManager().isProtectedBroadcast(
intent.getAction())) {
String msg = "Permission Denial: not allowed to send broadcast "
+ intent.getAction() + " from pid="
+ callingPid + ", uid=" + callingUid;
Slog.w(TAG, msg);
throw new SecurityException(msg);
}
} catch (RemoteException e) {
Slog.w(TAG, "Remote exception", e);
return BROADCAST_SUCCESS;
}
}
{% endhighlight %}
下面一段代码是对Stickyboradcast的处理。
步骤如下：
1. 检查权限.   2. 把这个广播放到stickybroadcast列表里等待触发。
{% highlight java %}
// Add to the sticky list if requested.
//Stickybroadcast的处理。
if (sticky) {
if (checkPermission(android.Manifest.permission.BROADCAST_STICKY,
callingPid, callingUid)
!= PackageManager.PERMISSION_GRANTED) {
String msg = "Permission Denial: broadcastIntent() requesting a sticky broadcast from pid="
+ callingPid + ", uid=" + callingUid
+ " requires " + android.Manifest.permission.BROADCAST_STICKY;
Slog.w(TAG, msg);
throw new SecurityException(msg);
}
if (requiredPermission != null) {
Slog.w(TAG, "Can't broadcast sticky intent " + intent
+ " and enforce permission " + requiredPermission);
return BROADCAST_STICKY_CANT_HAVE_PERMISSION;
}
if (intent.getComponent() != null) {
throw new SecurityException(
"Sticky broadcasts can't target a specific component");
}
//把要发出去的广播，保存的mStickyBroadcasts里面，用于register的时候，查询立即返回。
ArrayList<Intent> list = mStickyBroadcasts.get(intent.getAction());
if (list == null) {
list = new ArrayList<Intent>();
mStickyBroadcasts.put(intent.getAction(), list);
}
int N = list.size();
int i;
for (i=0; i<N; i++) {
if (intent.filterEquals(list.get(i))) {
// This sticky already exists, replace it.
list.set(i, new Intent(intent));
break;
}
}
if (i >= N) {
list.add(new Intent(intent));
}
}
{% endhighlight %}
下面的代码是获取能接受的这个广播的receiver。步骤是：
1. 先从packagemanager里获取（我们都知道，packagemanager保存了一个解析所有package的信息的两个文件，每次都从这个文件里加载信息，packagemanager）。
2. 从获取动态注册的receiver。（动态注册的receiver都保存在mReceiverResolver里面）
{% highlight java %}
// Figure out who all will receive this broadcast.
List receivers = null;
List<BroadcastFilter> registeredReceivers = null;
try {
if (intent.getComponent() != null) {
// Broadcast is going to one specific receiver class...
//通过packageManagerService来获取注册的receiver.很明显，是写在AndroidManifest.xml里的
ActivityInfo ai = AppGlobals.getPackageManager().
getReceiverInfo(intent.getComponent(), STOCK_PM_FLAGS);
if (ai != null) {
receivers = new ArrayList();
ResolveInfo ri = new ResolveInfo();
ri.activityInfo = ai;
receivers.add(ri);
}
} else {
// Need to resolve the intent to interested receivers...
//这里也是来获取注册在AndroidManifest.xml里的receiver
if ((intent.getFlags()&Intent.FLAG_RECEIVER_REGISTERED_ONLY)
== 0) {
receivers =
AppGlobals.getPackageManager().queryIntentReceivers(
intent, resolvedType, STOCK_PM_FLAGS);
}
//这里是用来获取，动态注册的receiver
registeredReceivers = mReceiverResolver.queryIntent(intent, resolvedType, false);
}
} catch (RemoteException ex) {
// pm is in same process, this will never happen.
}
final boolean replacePending =
(intent.getFlags()&Intent.FLAG_RECEIVER_REPLACE_PENDING) != 0;

if (DEBUG_BROADCAST) Slog.v(TAG, "Enqueing broadcast: " + intent.getAction()
+ " replacePending=" + replacePending);
{% endhighlight %}
如果不是order广播，先触发动态注册的receiver。然后动态的receiver清零。
{% highlight java %}
int NR = registeredReceivers != null ? registeredReceivers.size() : 0;
//如果不是order广播，先把触发动态注册的receiver
if (!ordered && NR > 0) {
// If we are not serializing this broadcast, then send the
// registered receivers separately so they don't wait for the
// components to be launched.
BroadcastRecord r = new BroadcastRecord(intent, callerApp,
callerPackage, callingPid, callingUid, requiredPermission,
registeredReceivers, resultTo, resultCode, resultData, map,
ordered, sticky, false);
if (DEBUG_BROADCAST) Slog.v(
TAG, "Enqueueing parallel broadcast " + r
+ ": prev had " + mParallelBroadcasts.size());
boolean replaced = false;
if (replacePending) {
for (int i=mParallelBroadcasts.size()-1; i>=0; i--) {
if (intent.filterEquals(mParallelBroadcasts.get(i).intent)) {
if (DEBUG_BROADCAST) Slog.v(TAG,
"***** DROPPING PARALLEL: " + intent);
mParallelBroadcasts.set(i, r);
replaced = true;
break;
}
}
}
if (!replaced) {
mParallelBroadcasts.add(r);
scheduleBroadcastsLocked();
}
registeredReceivers = null;
NR = 0;
}
{% endhighlight %}
否则把动态注册的和静态注册的合并，然后触发。
这里面的隐含的逻辑是：如果不是order广播，先触发动态监听的，然后触发静态监听的（不存在合并的问题，因为动态注册的列表已经清零）。
如果是order广播，合并后再顺序触发。具体动态监听和静态监听的具体的合并逻辑代码见下。
{% highlight java %}
// Merge into one list.
int ir = 0;
if (receivers != null) {
// A special case for PACKAGE_ADDED: do not allow the package
// being added to see this broadcast. This prevents them from
// using this as a back door to get run as soon as they are
// installed. Maybe in the future we want to have a special install
// broadcast or such for apps, but we'd like to deliberately make
// this decision.
String skipPackages[] = null;
if (Intent.ACTION_PACKAGE_ADDED.equals(intent.getAction())
|| Intent.ACTION_PACKAGE_RESTARTED.equals(intent.getAction())
|| Intent.ACTION_PACKAGE_DATA_CLEARED.equals(intent.getAction())) {
Uri data = intent.getData();
if (data != null) {
String pkgName = data.getSchemeSpecificPart();
if (pkgName != null) {
skipPackages = new String[] { pkgName };
}
}
} else if (Intent.ACTION_EXTERNAL_APPLICATIONS_AVAILABLE.equals(intent.getAction())) {
skipPackages = intent.getStringArrayExtra(Intent.EXTRA_CHANGED_PACKAGE_LIST);
}
if (skipPackages != null && (skipPackages.length > 0)) {
for (String skipPackage : skipPackages) {
if (skipPackage != null) {
int NT = receivers.size();
for (int it=0; it<NT; it++) {
ResolveInfo curt = (ResolveInfo)receivers.get(it);
if (curt.activityInfo.packageName.equals(skipPackage)) {
receivers.remove(it);
it--;
NT--;
}
}
}
}
}
int NT = receivers != null ? receivers.size() : 0;
int it = 0;
ResolveInfo curt = null;
BroadcastFilter curr = null;
while (it < NT && ir < NR) {
if (curt == null) {
curt = (ResolveInfo)receivers.get(it);
}
if (curr == null) {
curr = registeredReceivers.get(ir);
}
if (curr.getPriority() >= curt.priority) {
// Insert this broadcast record into the final list.
receivers.add(it, curr);
ir++;
curr = null;
it++;
NT++;
} else {
// Skip to the next ResolveInfo in the final list.
it++;
curt = null;
}
}
}
while (ir < NR) {
if (receivers == null) {
receivers = new ArrayList();
}
receivers.add(registeredReceivers.get(ir));
ir++;
}
if ((receivers != null && receivers.size() > 0)
|| resultTo != null) {
BroadcastRecord r = new BroadcastRecord(intent, callerApp,
callerPackage, callingPid, callingUid, requiredPermission,
receivers, resultTo, resultCode, resultData, map, ordered,
sticky, false);
if (DEBUG_BROADCAST) Slog.v(
TAG, "Enqueueing ordered broadcast " + r
+ ": prev had " + mOrderedBroadcasts.size());
if (DEBUG_BROADCAST) {
int seq = r.intent.getIntExtra("seq", -1);
Slog.i(TAG, "Enqueueing broadcast " + r.intent.getAction() + " seq=" + seq);
}
boolean replaced = false;
if (replacePending) {
for (int i=mOrderedBroadcasts.size()-1; i>0; i--) {
if (intent.filterEquals(mOrderedBroadcasts.get(i).intent)) {
if (DEBUG_BROADCAST) Slog.v(TAG,
"***** DROPPING ORDERED: " + intent);
mOrderedBroadcasts.set(i, r);
replaced = true;
break;
}
}
}
if (!replaced) {
mOrderedBroadcasts.add(r);
scheduleBroadcastsLocked();
}
}
return BROADCAST_SUCCESS;
{% endhighlight %}
两个列表从头到尾来比较（已静态注册为最终链表－－注意静态注册的列表的顺序本身就无法确定）：
如果碰到，动态注册的那一个priority高，那么，就把动态注册那个插入到静态注册列表里面，动态注册和静态注册的指针都下移（实际上，静态注册里要比较的那个不变）；如果静态注册注册的那个高，静态注册的指针不变，动态注册的指针下移。

看起来，这个操作是顺序排序的，但实际上，静态注册的列表和动态注册的列表本身的顺序就不是按照priority来的？