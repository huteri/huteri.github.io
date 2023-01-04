---

layout: post

title: "Common Mistakes When Doing Clean Architecture in Android"

date: 2017-08-08 16:21:47 +0800

comments: true

categories: clean architecture

---

It's been a while since I last wrote. Let's talk about clean architecture in Android. We know that best practices for clean architecture are changing fast. Look on what Google has done with its architecture components. For more detail about architecture components, [click here](https://developer.android.com/topic/libraries/architecture/index.html)

<!--more-->

But, it's not what I want to talk today. Let's talk more about common mistakes when doing clean archictecture in Android.

### 1. Not able to move logic from Activity/Fragment to Presenter/ViewModel

I believe that the view must be passive, which means that we should make it as dumb as possible so that there is no way for bugs to occur in the view code. Every method in a view should do exactly one thing, with no additional logic.

Let's look at a good example.


``` java

interface LoginView extends BaseView {

    void showPasswordInvalid();

    void showUsernameRequired();

    void navigateToMainPage();

    void showLoginError();

    void showLoginSucceed();
    
    void showInputOtpDialog();
    
    void showAlreadyLogin();

}

```

Basically, a method is verbose enough for us to understand so there is no room for error. but let's see some of the mistakes to give us a picture

``` java

interface LoginView extends BaseView {

    void showError(Error error);
    
    void navigateToActivity(Activity activity)

}

```

Now, imagine the implementation, the methods have more than 1 responsibility. `showError(Error error)` method has to parse the error first, and then decide how to show the error. it's more than 1 responsibility. 

Then came a question, the good example above shows us that we have to write `Toast.makeText` for every method that needs to show error message. it breaks the principle of DRY (don't repeat yourself), yes, and no. Yes, we will have same functionality for every method, and no, we can improve it by creating new method in the view to show toast. 

I know it's lot of boilerplate, but it's worth it if we think about how easy it will be to create unit test with verbose methods.

###2. Not to Think about Unit Test

Unit test is the most useful way to make our code clean. Even if we just think about it, it will improve our code a lot. One tip is to always think about unit testing while writing code. Think about how we are going to test it later. This is the least minimum, I mean if you do TDD, then it's better. No need to follow this.


Basically, by only thinking about unit test, we will: 

#### a. not put android framework code in the presenter/viewmodel

#### b. not put logic in activity/fragment because we are trying to achieve 100% coverage of the logic

#### c. not put complex code, but instead make everything simple

Let me know what you think later after you tried it.

###3. Not to Think About Modularity/Separation


One of the thing about clean architecture is separation. Every layer of clean architecture has their own responsibility and they only know layers below them, they only want to know that they can get the data they need. 

With this separation, we can change the inner logic of one layer without affecting the other layer as long as it can still get the data it needs. One of the useful example is local database. There are a lot of ways implementing local database in android such as Realm, Room, and Sqlite, and this thing will change to something better in the future. So if we already implemented Realm, then how would we change it to room later? 

This is where separations looks useful. With a clear separation, we could change to room easily without too much refactoring.

So, always think about separation in mind, think about how we could change the library to something better in the future, think about maintainability over features.

If you think there is more, add in the comment section below.







