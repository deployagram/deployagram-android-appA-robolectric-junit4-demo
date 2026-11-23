# Deployagram Android AppA Robolectric JUnit 4 Demo

The same small Android app as [`deployagram-android-appA-emulator-kotest-demo`](../deployagram-android-appA-emulator-kotest-demo): AppA has one
screen. The user types an email and presses **Recommend book**, and the app asks BookOverflow for a book suggestion.

This version tests the screen on the JVM, with **Robolectric** and **JUnit 4**. It needs no emulator, so it runs on any
machine or CI runner that has Java 21. The Kotest version runs on an emulator.

## What it does

AppA calls `GET /BookOverflow/suggestion?email=<email>` and shows the answer:

| Situation                                      | What the screen shows                                        |
|------------------------------------------------|--------------------------------------------------------------|
| BookOverflow suggests a book                   | The book's name, edition, format and authors                 |
| Email is blank, and BookOverflow refuses (400) | `You must enter a valid email!`                              |
| Any other failure                              | `Sorry, no suggestion right now (<reason>)`                  |
| Load Config, after a suggestion                | AliceMessager's reply, `World` for `Hello` (sent once a book is recommended) |
| Load Config, with no suggestion yet            | AliceMessager's reply, `Is someone there?` for an empty payload |

## What it is for

* drive an Android screen from a JVM test with **Espresso** and **Robolectric**;
* stand in for BookOverflow with a **MockWebServer** double, and check what the app sent;
* write the test in plain **JUnit 4**, the default for Android.

The test never sends an HTTP request. It types an email and presses the button, and AppA makes the request.

## Tech

* Kotlin, Android Gradle Plugin 8.9.0, Gradle 8.12 (checked-in wrapper)
* Java 17 to build the app, and Java 21 to run the tests (Robolectric's Android 16 runtime needs Java 21)
* Min SDK 26, compile and target SDK 36
* Robolectric 4.18.1, JUnit 4.13.2, Espresso 3.6.1, AndroidX Test core 1.6.1
* OkHttp 5.3.2 in the app, MockWebServer 5.3.2 in the tests

## Run the tests

```bash
./gradlew testDebugUnitTest
```

The test report is in `app/build/reports/tests/testDebugUnitTest/index.html`.

## Project layout

```
app/src/main/java/com/deployagram/demo/appa/
    MainActivity.kt          the one screen: email field, button, result
    BookOverflowClient.kt    GET /BookOverflow/suggestion, with OkHttp
    BookSuggestion.kt        the suggestion, and how it is shown
    BookOverflowConfig.kt    where BookOverflow is (the test changes it)
app/src/test/java/com/deployagram/demo/appa/
    RecommendBookScreenTest.kt   the two JVM tests, and the BookOverflow double
    LoadConfigButtonTest.kt      the two JVM tests of Load Config, and the AliceMessager double
```

## Deployagram instrumentation

The tests are instrumented with two Deployagram packages:

* `deployagram-junit4`: `DeployagramRule`, a JUnit 4 rule that ends each test's sequence diagram in the Collector. It is
  declared on the test class, because Robolectric runs on the JVM and has no run-level listener to register.
* `deployagram-mockwebserver`: `RecordingDispatcher`, which wraps the BookOverflow double. It records the request AppA
  sends, and the answer BookOverflow gives.

The Collector runs on the host, and the tests reach it at `http://localhost:1152`, so no emulator address is needed. The
caller and callee are named as in the Java repo: `AliceA` calls `ORG_ctm_01m1j08sx96792d2tysabvyadp:BookOverflow`.

```
  RecommendBookScreenTest (Robolectric) ──► AppA ──► RecordingDispatcher ──► BookOverflow double
                                                             │
                                                             ▼
                                               Collector on localhost:1152
```

### Running

The Deployagram packages are on Maven Central, so the build needs no local install. Start the Collector, with the licence
it needs, and run the tests:

```bash
./gradlew testDebugUnitTest
```

Each test's entries are under its own marker, `RecommendBookScreenTest.<test name>`.
