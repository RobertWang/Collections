> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/swlh/flutter-vs-native-vs-react-native-examining-performance-31338f081980#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImY0MTk2YWVlMTE5ZmUyMTU5M2Q0OGJmY2ZiNWJmMDAxNzdkZDRhNGQiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2MzQxNDQ5NTEsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjExMDk1MjQyMTYyNzIyMjczOTI0NCIsImVtYWlsIjoiY253YW5neWxAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImF6cCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsIm5hbWUiOiLnjovli4fosIUiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EtL0FPaDE0R2g2RnJpTUQzdDFhWk96a0tPYm1PT2FsbE5zX0dIck5HR2VvdEFQUXc9czk2LWMiLCJnaXZlbl9uYW1lIjoi5YuH6LCFIiwiZmFtaWx5X25hbWUiOiLnjosiLCJpYXQiOjE2MzQxNDUyNTEsImV4cCI6MTYzNDE0ODg1MSwianRpIjoiMjhhMjgxY2NmODRjMzMzN2Y5ZTAyZTY4NmFjYmQ4MGM5Mjc5MzIxNiJ9.jbyfFVc4TENBG4uPOWyijPH7k8CZU41B7cf7di6xJze4h9uLAJGyZrvk33cEylSqyCpDQDQ4R_Q-SriqRBl-kVJbg0f0u9vqVacZb05NrkIAHqDpx8ovofcKX_pch8fuDynN1yQ8Fvld1oYx4Dain3Ibi7YlbmvCqiDh3rBCKTN3KGolz2o73jD-s6tLcmDxOLSUwOR9uHddy4eAr2RS5yuXTaO-UssPLVuiRNVnpiTH2hf8rO8uYz51x2gXtnMds5weGJug_F7LVIQWRjzrpJXV-fzKWV0JUZFzAK52Gtv5rwwmPMVHNwXViu6CIW8lts8H_6FRLCHbGZRm33uMkw)

> Today some of the most popular solutions to build mobile apps are native or cross-platform approaches......

[![](https://miro.medium.com/fit/c/96/96/2*_ufZYLoBvvfqNk_m9CuLnw.png)](https://medium.com/@inverita?source=post_page-----31338f081980--------------------------------)

![](https://miro.medium.com/max/2000/1*_5uHflEhilD6S0cvr6wzPw.jpeg)

Today some of the most popular solutions to build mobile apps are native or cross-platform approaches using React Native or [Flutter](https://inveritasoft.com/technologies/flutter). While native development is positioned as AAA technical solution, it has some disadvantages that create market space for cross-platform apps to come in. In general, native development requires more effort from the development team to accomplish the project but it gives full control over tricky technical stuff under the hood. On the other hand, if you choose cross-platform, it can significantly speed up the development process due to a common code base, make project support easier and reduce expenses for development.

One more advantage of native over cross-platform development is performance. In the technical world, you can encounter “cross-platform apps are slow” stereotypes. We decided to test if it’s true and to what extend cross-platform apps are slower than native.

1.  Interacting with phone API (accessing photos, file system, getting GPS location and so on).
2.  Rendering speed (animation smoothness, frames per second while UI is changed or some UI effects that take place in time).
3.  Business logic (the speed of mathematical calculations and memory manipulations. This type of performance is most important for the apps with complex business logic).

In this article, we share the results of performance tests showing mathematical calculations of number Pi implemented in native and cross-platform approaches.

![](https://miro.medium.com/max/1400/1*Ey1f_IcZiwM75XGKY83nVw.jpeg)

Memory-intensive test (Gauss–Legendre algorithm) for iOS

**iOS**

*   Objective-C is the best programming language for iOS development. Swift is 1.7 times slower compared to Objective C.
*   Surprise: Flutter is a bit faster than Swift (on 15%).
*   React Native is 20 times slower than Objective C.

![](https://miro.medium.com/max/60/1*wHGAVXahLFWFPRrzmYN6fw.jpeg?q=20)

![](https://miro.medium.com/max/1400/1*wHGAVXahLFWFPRrzmYN6fw.jpeg)

CPU-intensive test (Borwein algorithm) for iOS

**iOS**

*   Objective C is the best option for iOS app development. Swift is 1.9 times slower compared to Objective-C.
*   Flutter is 5 times slower than Swift.
*   React Native version is more than 15 times slower than the Swift version.

![](https://miro.medium.com/max/60/1*C5bM9KdtdjHpftFxgBn-UA.jpeg?q=20)

![](https://miro.medium.com/max/1400/1*C5bM9KdtdjHpftFxgBn-UA.jpeg)

Memory-intensive test (Gauss–Legendre algorithm) for Android

**Android**

*   Java and Kotlin have similar performance indications and are the best options for Android development.
*   Flutter is approximately 20% slower than native.
*   React Native is around 15 times slower than native.

![](https://miro.medium.com/max/60/1*zf1pnAPXytzFvlqj_aDThA.jpeg?q=20)

![](https://miro.medium.com/max/1400/1*zf1pnAPXytzFvlqj_aDThA.jpeg)

CPU-intensive test (Borwein algorithm) for Android

**Android**

*   Java and Kotlin have similar performance indications and are the best options for Android development.
*   Native is 2 times faster then Flutter.
*   React native is around 6 times slower than native.

1.  All tests have been done on real physical devices (iPhone 6s IOS 13.2.3 and Xiaomi Redmi Note 5 running under Android 9.0);
2.  We measured performance on release builds. In some cases, debug builds can be significantly slower than the release builds.
3.  All tests were run several times and the average result was calculated.
4.  Gauss–Legendre & Borwein algorithms of calculating Pi numbers were used. The Pi number has been calculated 100 times with 10 million digits precision.
5.  Gauss–Legendre is a more memory-intensive algorithm in comparison with Borwein, but Borwein is more CPU-intensive.
6.  [Source code](https://github.com/nazarcybulskij/Mobile_Bechmarks_)

1.  In summary, not all cross-platform apps are slow. What’s more than that, Flutter apps have higher performance than Swift apps.
2.  Objective C and Flutter will be a wise choice if you want to develop a super-fast iOS app.
3.  For the apps with high load calculations Flutter is a good option for both, Android and iOS app development.

Please let [inVerita](https://inveritasoft.com/blog/flutter-vs-native-vs-react-native-examining-performance) know if you struggle with picking a mobile tool for development, always happy to help.