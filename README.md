# UnknowApp
### 完美的适配Android8.0未知来源应用安装权限方案


2018年5月纠正：
[@this蜗牛](https://my.csdn.net/a23006239) 提供的需要设置包名，去打开权限设置界面才能在onActivityResult中接收到【resultCode 等于 RESULT_OK 】

所以修改方法：
```
	@RequiresApi(api = Build.VERSION_CODES.O)
    private void startInstallPermissionSettingActivity() {
	    //注意这个是8.0新API
	    Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
        startActivityForResult(intent, 10086);
    }
```

为如下：
```
	@RequiresApi(api = Build.VERSION_CODES.O)
    private void startInstallPermissionSettingActivity() {
	    Uri packageURI = Uri.parse("package:" + getPackageName());
	    //注意这个是8.0新API
	    Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES, packageURI);
        startActivityForResult(intent, 10086);
    }
```
感谢 [@this蜗牛](https://my.csdn.net/a23006239) 提出的意见

--------
Android8.0的诸多新特性中有一个非常重要的特性：未知来源应用权限

以前安装未知来源应用的时候一般会弹出一个弹窗让用户去设置允许还是拒绝，并且设置为允许之后，所有的未知来源的应用都可以被安装。

Android8.0的变化是，未知应用安装权限的开关被除掉，取而代之的是未知来源应用的管理列表，需要在里面打开每个应用的未知来源的安装权限。Google这么做是为了防止一开始正经的应用后来开始通过升级来做一些不合法的事情，侵犯用户权益。
当你的应用直接适配到Android8之后，内部启动应用安装是会被阻塞的，如果不处理好这个未知来源的权限，会导致应用根本无法更新，只能去应用市场重新下载。
那么如何来适配8.0这一个新变化呢？

 1、在清单文件中增加请求安装权限 

```
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
```
2、我们还需要在代码里面对权限进行处理
首先用canRequestPackageInstalls()方法判断你的应用是否有这个权限
```
haveInstallPermission = getPackageManager().canRequestPackageInstalls();
```

如果haveInstallPermission 为 true，则说明你的应用有安装未知来源应用的权限，你直接执行安装应用的操作即可。

如果haveInstallPermission 为 false，则说明你的应用没有安装未知来源应用的权限，则无法安装应用。由于这个权限不是运行时权限，所以无法再代码中请求权限，还是需要用户跳转到设置界面中自己去打开权限。
#####具体的操作是：
弹出dialog，告知用户

> "安装应用需要打开未知来源权限，请去设置中开启权限"

然后用户点击确定之后跳转到未知来源应用权限管理列表：
```java 
Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
startActivityForResult(intent, 10086);
```
然后在onActivityResult中去接收结果：

```java 
if (resultCode == RESULT_OK && requestCode == 10086) {
            installProcess();//再次执行安装流程，包含权限判等
 }
```

##### 完整的流程如下：
```java 
	//安装应用的流程
	private void installProcess() {
        boolean haveInstallPermission;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
	        //先获取是否有安装未知来源应用的权限
            haveInstallPermission = getPackageManager().canRequestPackageInstalls();
            if (!haveInstallPermission) {//没有权限
                DialogUtils.showDialog(this, "安装应用需要打开未知来源权限，请去设置中开启权限", 
                new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                            startInstallPermissionSettingActivity();
                        }
                    }
                }, null);
                return;
            }
        }
		//有权限，开始安装应用程序
        installApk(apk);
    }
    
    
	@RequiresApi(api = Build.VERSION_CODES.O)
    private void startInstallPermissionSettingActivity() {
	    Uri packageURI = Uri.parse("package:" + getPackageName());
	    //注意这个是8.0新API
	    Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES, packageURI);
        startActivityForResult(intent, 10086);
    }

	//安装应用
	private void installApk(File apk) {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N) {
            intent.setDataAndType(Uri.fromFile(apk), "application/vnd.android.package-archive");
        } else {//Android7.0之后获取uri要用contentProvider
            Uri uri = AppCommonUtils.getUriFromFile(getBaseContext(), apk);
            intent.setDataAndType(uri, "application/vnd.android.package-archive");
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        }
        
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        getBaseContext().startActivity(intent);
    }
```


#### 以上，Android8.0的未知来源应用安装权限适配基本结束，希望对大家能有帮助





### 

[转载请注明出处](http://blog.csdn.net/changmu175/article/details/78906829)
-----
