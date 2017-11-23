需要关心的几个问题：

1. 用户信息的缓存，包括：缓存方式，缓存了哪些信息，缓存更新时机，
2. 单点登录，实现方式
3. 首次登录  和 第二次使用Token的登录


==== 用户信息 UserInfo 

BootActivity -----> redirectTo() ----> 

获取Token，如果Token不为空，就处理登录逻辑；否则，去登录页面。

```
private void redirectTo() {

    String accessToken = TinkerManager.getApplicationLike().getUserToken();
    if (StringUtils.isNotEmpty(accessToken)) {
        if (mApp.getUserInfo() != null && mApp.isInvestor() && mApp.getUserInfo().isShowLicense()) {
            UIHelper.startUserProfileActivity(BootActivity.this, "", "");
            finish();
        } else {
            UIHelper.goMain(BootActivity.this, getIntent());
        }
    } else {
        Bundle bundle = null;
        if (getIntent() != null) {
            bundle = getIntent().getExtras();
        }
        UIHelper.startActivity(this, UserLoginMainActivity.class, bundle, true);
    }
}
```

getUserInfo======

```
public UserInfo getUserInfo() {
    if (mUserInfo == null) {
        String data = PreferenceUtils.readStrConfig(com.ethercap.base.android.application.Constants.PreferKeys.KEY_USER_INFO, getApplication(), "");
        mUserInfo = GsonUtil.jsonToObject(UserInfo.class, data);

    }
    return mUserInfo;
}
```

=======LoginMain

登录方式包括： 微信登录，账号密码登录

用户信息的缓存：

```
public void setUserInfo(UserInfo info) {
    if (info == null) {
        return;
    }
    mUserInfo = info;
    PreferenceUtils.writeStrConfig(com.ethercap.base.android.application.Constants.PreferKeys.KEY_USER_INFO,
   GsonUtil.objectToJson(mUserInfo), getApplication());
}
```

疑问：

1. 为什么不在登录的时候，直接将用户信息返回，而要绕道拿着Token再去拿用户信息？



======== 单点登录====



