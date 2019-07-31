Members of the Flutter team are working to bring full macOS Catalina support to Flutter. Work is being track on the GitHub project [Flutter on macOS Catalina Support](https://github.com/orgs/flutter/projects/4?add_cards_query=is%3Aopen). In the mean time, we know that some of you have wanted to use the macOS Catalina betas with Flutter. Here’s [christopherfujino](http://www.github.com/christopherfujino)’s step-by-step diary of what he’s experienced getting it working, which is helping inform the team’s work as to what’s necessary.

## Known Issues

1. Gatekeeper preventing “kernel-service.dart.snapshot” from executing [#36714](https://github.com/flutter/flutter/issues/36714) **Workaround**: `sudo spctl --master-disable`
2. [Xcode] Cannot build 32-bit APK on XCode >= 10 [flutter #36114](https://github.com/flutter/flutter/issues/36114) [Dart #36839](https://github.com/dart-lang/sdk/issues/36839) **Workaround**: install Xcode version 9
3. Website will need to be updated to illustrate how to update path for zsh (Catalina’s default shell) [website #2856](https://github.com/flutter/website/issues/2856) **Workaround**: `echo 'export PATH="[PATH_TO_FLUTTER_GIT_DIRECTORY]/bin:$PATH"' >> $HOME/.zshrc`
4. Cocoapod path broken by upgrading system ruby version [#36786](https://github.com/flutter/flutter/issues/36786) **Workaround**: reinstall cocoapods, e.g. `sudo gem install cocoapods`.
5. [Xcode] Update website documentation to reflect new location in Xcode of how to configure signing certificate [website #2857](https://github.com/flutter/website/issues/2857)
6. [Xcode] For each flutter app that I tried to run on an iOS device, I had to add my signing certificate in XCode [#36788](https://github.com/flutter/flutter/issues/36788)

## OS Setup

1. This was on a new Mac Mini, right from the Apple Store
1. Upgraded OS to Catalina preview, via [https://www.apple.com/macos/catalina-preview/](https://www.apple.com/macos/catalina-preview/)
1. On macOS Catalina v10.15 Beta (19A512f), XCode Beta 11.0
1. On iterm2

## Journal

1. Downloaded stable flutter sdk from flutter.dev, v1.7.8+hotfix.3
2. OS X prompts if iterm2 should be allowed access to `$HOME/Downloads`
3. When I enter the command `flutter/bin/flutter doctor`, OS X says “dart” can’t be opened because it was not downloaded from the App Store
    1. After going to Settings -> Security & Privacy -> Open Anyway, I get another dialog: `dart` is a Unix app downloaded from the Internet. Are you sure you want to open it?
    2. On clicking “open”, a new “Terminal” window was opened, and then it executes just `dart`, without any arguments, and then the terminal session exits.
4. When I re-issue the same command, “`flutter/bin/flutter doctor`”, this time the actual flutter tool is run, however a new dialog is opened: “kernel-service.dart.snapshot” can’t be opened because Apple cannot check it for malicious software.
    3. When I go to Settings -> Security & Privacy -> Open Anyway, nothing happens. The text next to the button says: `kernel-service.dart.snapshot` was blocked from opening because it is not from an identified developer. Re-running the command results in no change.
5. Trying to switch channels failed with: 
```
flutter@macN1 flutter % bin/flutter channel master
Switching to flutter channel 'master'...
../../third_party/dart/runtime/bin/snapshot_utils.cc: 149: error: Failed to memory map snapshot: /Users/flutter/flutter/bin/cache/dart-sdk/bin/snapshots/kernel-service.dart.snapshot version=2.4.0 (Wed Jun 19 11:53:45 2019 +0200) on "macos_x64"
thread=5891, isolate=(null)(0x0)
 pc 0x000000010bd945d5 fp 0x0000700006ee7c40 dart::Profiler::DumpStackTrace(void*)
 pc 0x000000010badfdc2 fp 0x0000700006ee7d20 dart::Assert::Fail(char const*, ...)
 pc 0x000000010bac2dfb fp 0x0000700006ee7dd0 dart::bin::Snapshot::TryReadAppSnapshot(char const*)
 pc 0x000000010bac7eea fp 0x0000700006ee7e50 dart::bin::CreateIsolateAndSetup(char const*, char const*, char const*, char const*, Dart_IsolateFlags*, void*, char**)
 pc 0x000000010bcdba48 fp 0x0000700006ee7f00 dart::RunKernelTask::Run()
 pc 0x000000010be36a0d fp 0x0000700006ee7f30 dart::ThreadPool::Worker::Loop()
 pc 0x000000010be36880 fp 0x0000700006ee7f70 dart::ThreadPool::Worker::Main(unsigned long)
 pc 0x000000010bd90fc1 fp 0x0000700006ee7fb0 dart::ThreadStart(void*)
 pc 0x00007fff73470cce fp 0x0000700006ee7fd0 _pthread_start
 pc 0x00007fff7346d72b fp 0x0000700006ee7ff0 thread_start
-- End of DumpStackTrace
bin/flutter: line 183:  8210 Abort trap: 6           "$DART" --packages="$FLUTTER_TOOLS_DIR/.packages" $FLUTTER_TOOL_ARGS "$SNAPSHOT_PATH" "$@"
```
6. Following [this article](https://www.imore.com/how-open-apps-unidentified-developers-mac) and [this comment](https://github.com/flutter/flutter/issues/33890#issuecomment-505160047), I disabled Gatekeeper globally, so that binaries can execute without being signed. Now flutter doctor no longer throws the dialog about “kernel-service.dart.snapshot” being unsigned. I was also able to successfully switch channels to master. [Issue](https://github.com/flutter/flutter/issues/36714)
7. Under “flutter doctor” (FlutterValidator), “Downloaded executables cannot execute on host”. [Issue](https://github.com/flutter/flutter/issues/22598)
8. We will have to update website documentation on updating path to reflect that Catalina will default to zsh. I updated my path in $HOME/.zshrc. [Issue](https://github.com/flutter/website/issues/2856)
9. I don’t know how to install xcode. Tried the app store, I’m getting weird hits, including tutorials and xcode plugins, but I don’t see an actual xcode install...so it turns out the App Store version of Xcode doesn’t support Catalina. I needed to install the beta version from [https://developer.apple.com/download](https://developer.apple.com/download)
10. Even after installing the Xcode beta, flutter doctor can’t detect it. The instructions “sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer” don’t work because xcode isn’t at that location. **Fixed**: .app file was in $HOME/Downloads/
11. [Cocoapods not detected](https://paste.googleplex.com/5337691329658880), looks like [this issue](https://github.com/CocoaPods/CocoaPods/issues/6898). **Fixed** with “sudo gem install -n /usr/local/bin cocoapods”. Ran “pod setup” and cocoapods support detected. [Issue](https://github.com/flutter/flutter/issues/36786)
12. With ios 13 simulator, issued “flutter run”, it worked.
13. Attempted to run on ipod touch in debug:
    1. No valid code signing certs
    1. Following instructions, “open ios/Runner.xcworkspace” -> Xcode
    1. In Xcode 11.0 beta 4, the signing interface is no longer under “General”, but in its own category, “Signing & Capabilities” [Issue](https://github.com/flutter/website/issues/2857)
    1. Re-running “flutter run”, hit [this issue](https://github.com/flutter/flutter/issues/35989), however it doesn’t seem Catalina-dependent. “Flutter clean” fixed it.
14. It seems like for each new Flutter app to be run on an iOS device, the cert needs to be added in Xcode. [Issue](https://github.com/flutter/flutter/issues/36788)
15. Attempted to run on ipod touch in release:
    1. Successful
16. Downloaded Android Studio/SDK
    1. Intel HAXM installer was blocked by Gatekeeper
    1. “Flutter run” successful on emulator
17. Attempting to run on LG G7 ThinQ in debug:
    1. Successful
18. Attempting to run on LG G7 ThinQ in release:
    1. Successful
19. Run devicelab integration_ui_ios.dart test, passes
20. Build APK
    1. [Failure](https://paste.googleplex.com/4621858055913472) [Issue](#bookmark=id.7mbconxtxa5o)

## Links
1. [Official Apple Preview Website](https://www.apple.com/macos/catalina-preview/)
2. [All Github Catalina Issues](https://github.com/flutter/flutter/issues?q=is%3Aopen+is%3Aissue+label%3A%E2%80%8E%E2%80%8D%F0%9F%92%BBplatform-catalina)
3. [Article on security changes](https://www.cnet.com/news/6-macos-catalina-security-changes-from-apple-coming-this-fall/)
4. [Analytics on "run" usage of Catalina users vs total](https://analytics.google.com/analytics/web/#/report/app-content-pages/a67589403w118070018p123509034/explorer-table.plotKeys=%5B%5D&explorer-table.rowCount=10&_r.drilldown=analytics.contentDescription:run&_.useg=userXdHHlEdERVe-Do4CY8LIzA,builtin13&explorer-graphOptions.clearCompareConcept=true)