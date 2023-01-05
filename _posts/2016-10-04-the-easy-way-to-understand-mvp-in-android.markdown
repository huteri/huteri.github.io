---
layout: post
title: "The Easy Way to Understand MVP in Android"
date: 2016-03-25 14:12:18 +0700
comments: true
categories: architecture
---

Today, I would like to talk about MVP in android. There are a lot of resources recently talking about MVP in android. MVP is now considered as the good architecture available in Android Development. 

<!--more-->

I hope there is a better way to understand the concept of the MVP first before we talk about how to use MVP in our development workflow. This is the purpose of this article. I want to give a good and easy sample how to implement MVP. 

If you are also new in MVP, then I suggest to read this post first and then continue on the article I’ll give in the references.

So, the one of the idea of MVP is to make the code testable. There are not much details about how to write a good unit test in Android because unit testing was considered hard because of how dependable our project to android frameworks.

But, today, with MVP and clean architecture, we can improve the coverage of unit testing in our code.

Let’s see the code. I have written a simple android app for this purpose. It is mostly based on the references but I have improved it to make it easier to understand.

I didn't use any third party library for learning purpose. I also put some common best practices if you want to use it in real development.

Here is the link https://github.com/huteri/mvp-android

The thing about MVP is that we should not access the data layer (Model) directly from the view. We have to use the middle layer called presenter to coordinate the data into the view. But why? because view is heavily depended on android framework such as activity and fragment and it makes testing data layer in the view almost impossible. Data layer consists of our business logic and it should be tested properly.

**How do we do it?**

By moving business logic of the view to presenter. Now presenter is responsible for getting and managing the data. View just show it to UI by using android framework.

Now, we understand the concept, but how do we implement it? We have 2 steps here,

1. Delegate all the user actions from view to presenter
2. Presenter can send the data to the view and the view should show it to UI.

Let’s talk about the first step,

```java
package me.huteri.weather.features.main;

import android.os.Bundle;
import android.support.annotation.VisibleForTesting;
import android.support.v4.widget.SwipeRefreshLayout;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.Toast;

import java.util.List;

import me.huteri.weather.R;
import me.huteri.weather.features.BaseActivity;
import me.huteri.weather.features.adapter.WeatherListAdapter;
import me.huteri.weather.model.Weather;
import me.huteri.weather.model.service.WeatherApiImpl;


public class MainActivity extends BaseActivity implements MainView, WeatherListAdapter.WeatherItemListener {

    private MainPresenterImpl mPresenter;

    private WeatherListAdapter mWeatherListAdapter;
    private SwipeRefreshLayout mSrl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mPresenter = new MainPresenterImpl(this, new WeatherApiImpl());

        initRecyclerView();
        initSwipeRefreshLayout();
    }

    @Override
    protected void onResume() {
        super.onResume();
        mPresenter.loadWeatherData();

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.action_refresh:
                mPresenter.loadWeatherData();
                break;
        }
        return true;
    }

    @Override
    public void showProgress() {
        if(!mSrl.isRefreshing()) {

            // make sure setRefreshing() is called after the layout done everything else
            mSrl.post(new Runnable() {
                @Override
                public void run() {
                    mSrl.setRefreshing(true);
                }
            });
        }
    }

    @Override
    public void hideProgress() {
        if(mSrl.isRefreshing()) {
            mSrl.setRefreshing(false);
        }
    }

    @Override
    public void showWeatherClickedMessage(Weather s) {
        Toast.makeText(this, String.format(getString(R.string.main_toast_weather_item_click), s.getCityName()), Toast.LENGTH_SHORT).show();
    }

    @Override
    public void showWeathers(List<Weather> weathers) {
        mWeatherListAdapter.replaceData(weathers);

    }

    @Override
    public void showConnectionError() {
        Toast.makeText(this, R.string.main_error_connection, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onWeatherItemClick(Weather item) {
        mPresenter.clickWeatherItem(item);

    }

    private void initSwipeRefreshLayout() {

        mSrl = (SwipeRefreshLayout) findViewById(R.id.srl);
        mSrl.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                mPresenter.loadWeatherData();
            }
        });

    }

    private void initRecyclerView() {
        mWeatherListAdapter = new WeatherListAdapter(this, this);

        RecyclerView rvWeatherList = (RecyclerView) findViewById(R.id.rv_main);
        rvWeatherList.setLayoutManager(new LinearLayoutManager(this));
        rvWeatherList.setAdapter(mWeatherListAdapter);
    }
}
```

Here is how presenter looks like

```java
package me.huteri.weather.features.main;

import me.huteri.weather.model.Weather;

public interface MainPresenter{
    void loadWeatherData();
    void clickWeatherItem(Weather item);
}
```

```java
package me.huteri.weather.features.main;


import java.util.List;

import me.huteri.weather.features.BasePresenter;
import me.huteri.weather.model.Weather;
import me.huteri.weather.model.service.WeatherApi;
import me.huteri.weather.model.service.WeatherApiImpl;
import me.huteri.weather.util.EspressoIdlingResource;

/**
 * The implementation of the main presenter interface
 */

public class MainPresenterImpl extends BasePresenter implements MainPresenter {

    private final MainView mView;
    private final WeatherApiImpl mWeatherApi;

    public MainPresenterImpl(MainView view, WeatherApiImpl weatherApi) {
        mView = view;
        mWeatherApi = weatherApi;
    }

    @Override
    public void loadWeatherData() {

        mView.showProgress();

        EspressoIdlingResource.increment();

        mWeatherApi.getAllWeathers(new WeatherApi.WeatherServiceCallback<List<Weather>>() {

            @Override
            public void onSuccess(List<Weather> weathers) {
                EspressoIdlingResource.decrement();
                mView.hideProgress();
                mView.showWeathers(weathers);
            }

            @Override
            public void onFailure() {
                EspressoIdlingResource.decrement();
                mView.showConnectionError();
                mView.hideProgress();
            }
        });
    }

    @Override
    public void clickWeatherItem(Weather item) {
        mView.showWeatherClickedMessage(item);
    }
}
```

we define the method for `MainPresenter.theMethodHere()`, basically all the methods delegated from view. 

Now onto second step, presenter pass data to view, and the view show it to UI. We have `MainView` interface to do this

```java
package me.huteri.weather.features.main;

import java.util.List;

import me.huteri.weather.model.Weather;

public interface MainView {
    void showProgress();
    void hideProgress();
    void showWeatherClickedMessage(Weather s);
    void showWeathers(List<Weather> weathers);
    void showConnectionError();
}
```
And `MainActivity` only needs to implement this interface. We implement MainView interface because this is the way for the presenter to inform the view about things to show to UI side

This is only about view layer here.

With this separation, we can test our business logic easily without depending on android framework. 


```java
package me.huteri.weather.features.main;

import org.junit.Before;
import org.junit.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.ArrayList;
import java.util.List;

import me.huteri.weather.model.Weather;
import me.huteri.weather.model.service.WeatherApi;
import me.huteri.weather.model.service.WeatherApiImpl;

import static org.mockito.Mockito.verify;

public class MainPresenterImplTest {

    @Mock
    private MainView mMainView;

    @Mock
    private WeatherApiImpl mWeatherApi;

    /**
     * {@link ArgumentCaptor} is a powerful Mockito API to capture callback from background task and then perform assertion
     */
    @Captor
    private ArgumentCaptor<WeatherApi.WeatherServiceCallback> mWeatherServiceCallback;

    private MainPresenterImpl mMainPresenter;

    @Before
    public void setupMainPresenter() {
        MockitoAnnotations.initMocks(this);

        mMainPresenter = new MainPresenterImpl(mMainView, mWeatherApi);
    }

    @Test
    public void testLoadWeatherData_andSucceed() throws Exception {

        List<Weather> sample = generateSampleWeatherList();

        mMainPresenter.loadWeatherData();
        verify(mMainView).showProgress();
        verify(mWeatherApi).getAllWeathers(mWeatherServiceCallback.capture());

        mWeatherServiceCallback.getValue().onSuccess(sample);

        verify(mMainView).hideProgress();
        verify(mMainView).showWeathers(sample);

    }

    @Test
    public void testLoadWeatherData_andFailed() throws Exception {
        mMainPresenter.loadWeatherData();
        verify(mMainView).showProgress();

        verify(mWeatherApi).getAllWeathers(mWeatherServiceCallback.capture());
        mWeatherServiceCallback.getValue().onFailure();

        verify(mMainView).hideProgress();
        verify(mMainView).showConnectionError();
    }

    @Test
    public void testClickWeatherItem_showsWeatherMessage() throws Exception {
        Weather item = new Weather(22.1f, 3, "Jakarta", "Sample Description");

        mMainPresenter.clickWeatherItem(item);
        verify(mMainView).showWeatherClickedMessage(item);
    }

    private List<Weather> generateSampleWeatherList() {
        Weather item1 = new Weather(22.4f, 3, "Jakarta", "Cloudy");
        Weather item2 = new Weather(30.4f, 3, "Singapore", "Raining");

        List<Weather> list = new ArrayList<>();
        list.add(item1);
        list.add(item2);

        return list;
    }
}
```

We can use mockito to mock some dependency such as MainView interface and the data layer.

The full source code is available on [github](https://github.com/huteri/mvp-android)

**References**

https://codelabs.developers.google.com/codelabs/android-testing/index.html

http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/

https://www.youtube.com/watch?v=4L7G1uaQ67s

http://www.thedroidsonroids.com/blog/android/example-realm-mvp-dagger/

