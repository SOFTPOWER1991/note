排会入口：

项目详情页，右下角：代码如下：

```
//处理排会相关动作
private void handleMeetingAction(ProjectInfoDetail.PrivilegeData privilegeData) {
    if (privilegeData == null) {
        return;
    }
    if ("applyMeeting".equals(privilegeData.getType())) {
        doCreateMessageGroup(true, "", true);
    } else if ("modifyMeeting".equals(privilegeData.getType())) {
        doCreateMessageGroup(false, "修改会议", true);
    } else if ("feedback".equals(privilegeData.getType())) {
        //提交反馈
        mView.updateRefreshLayoutTag(true);
        if (mFeedbackMeetingInfoList.size() == 1) {
            UIHelper.startWebJsActivity(mContext, mFeedbackMeetingInfoList.get(0).getFeedbackUrl(), true);
        } else {
            Bundle bundle = new Bundle();
            bundle.putSerializable(Constants.BundleKeys.BUNDLE_KEY_EXTRA, mFeedbackMeetingInfoList);
            if (mDetailInfo != null && !TextUtils.isEmpty(mDetailInfo.getTitle())) {
                bundle.putString(Constants.BundleKeys.BUNDLE_KEY_PROJECT_TITLE, mDetailInfo.getTitle());
            }
            UIHelper.startActivity(bundle, Constants.RouterPath.MEETING_LIST_FOUNDER_MEETING_FEEDBACK_LIST, mContext);
        }
    }
}
```

Activity页面：

**MeetingArrangeActivity**

城市与开始时间：

**PickCityAndTimeActivity**

