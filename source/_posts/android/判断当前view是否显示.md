---
title: 判断当前view是否显示
date: 2018-04-13 22:34:53
categories: 实习日记
tags: android
---

最近有一个这样的需求，需要知道某个view的展示率，但是这个view嵌套在一个滑动布局中，在当前view已经被添加到滑动布局的前提下，该view也不一定会展示，可能会出现的情况如下所示：

![可能存在的情况.png](https://upload-images.jianshu.io/upload_images/2178834-a3e2d70d30d522f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么，我们怎么判断当前的view是否展示了呢？

我的实现思路如下：
1. 首先，在添加view的时候，判断当前view是否展示，如果展示，直接上报即可。如果没有展示，而是如图所示的情况，那就存储一个sp的值，用于表示当前的view没有上报。
2.在滑动布局中，进行滑动监听。在滑动的过程中，判断当前的view是否上报，在没有上报的情况下，继续判断view是否显示，如果在混动过程中view显示了，就上报view显示了，并存储一个sp的值，用于表示当前的view已经上报了。

上面的语言用代码表示为：
在添加view的逻辑中，用如下代码：
```
//addView代表该view被添加到当前布局了
  if (addView()) {
//parentView 即滑动布局
            parentView.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
                @Override
                public void onGlobalLayout() {
                    parentView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
//这里是判断当前布局是否可见
                    if (CustomMainScrollView.checkIsVisible(mContext, parentView)) {
                        reportSkinShareCard(gamemaster_app_cardshow_new.OP_VISIBLE, false);
                    }else{
                        //设置状态为未上报
PreferencesUtils.getInstance().putBoolean(HAS_REPORT_SHOW_SKIN_THEME_CARD, false);
                    }
                }
            });
        }
```
然后在滑动布局中，监听滚动事件，调用下面的代码：
```
 private void reportSkinThemeCard(Context context, View skinThemeView) {
        boolean hasReport = PreferencesUtils.getInstance().getBoolean(HAS_REPORT_SHOW_SKIN_THEME_CARD, false);

        if (!hasReport && checkIsVisible(context, skinThemeView) && mSkinThemeView.getChildCount() != 0 ) {
            byte  cardStyle = AbTestLogicManager.isSkinWithImageTestValue()?gamemaster_app_cardshow_new.CARD_TYPE_SKIN_WITH_IMAGE:gamemaster_app_cardshow_new.CRAD_SKIN_WITH_TEXT;

            // todo 在这里进行上报操作
            PreferencesUtils.getInstance().putBoolean(PreferenceConstants.HAS_REPORT_SHOW_SKIN_THEME_CARD, true);
//最后，更新状态，避免重复上报
        }
    }
```
在补充一个检查当前view是否显示的方法：
```
public static Boolean checkIsVisible(Context context, View view) {
        int screenWidth = getScreenMetrics(context).x;
        int screenHeight = getScreenMetrics(context).y;
        Rect rect = new Rect(0, 0, screenWidth, screenHeight);
        int[] location = new int[2];
        view.getLocationInWindow(location);
        if (view.getLocalVisibleRect(rect)) {
            return true;
        } else {
            return false;
        }
    }
```