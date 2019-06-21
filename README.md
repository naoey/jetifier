# jetifier

The jetifier AdnroidX transition tool in npm format, with a react-native compatible style

## Do you need this?

If you use React Native modules with native Java code that isn't converted to AndroidX, and your app is AndroidX, you probably need this.

Why?

The [standard AndroidX migration](https://developer.android.com/jetpack/androidx/migrate) rewrites your **current** installed source code, and at build time dynamically re-writes any linked jar/aar/zip files. This is all a "normal" Android app needs.

React Native apps are not standard Android apps. React Native modules with native Java code usually distribute that code as source, and link the source code directly. 

When you update your modules (or install them again after following the standard AndroidX migration), the freshly installed Java code from your react-native dependencies will not be translated to AndroidX anymore, and your build will fail.

So you have to perform an AndroidX migration on your linked source every time you update react native modules that ship native Java code. That is what this tool does - it can rewrite the source in node_modules every time you call it.

## Usage for source files

### To jetify / convert node_modules dependencies to AndroidX

Imagine you have a react-native project. [One of your library dependencies converts to AndroidX.](https://developers.google.com/android/guides/releases#june_17_2019), and you need to use the new version.

So now you need to convert your app to AndroidX, but many of your react-native libraries ship native Java code and have not updated. How can you do it?

1. Convert your app to AndroidX via the [standard AndroidX migration](https://developer.android.com/jetpack/androidx/migrate)
1. `npm install --save-dev jetifier` (or use yarn, but install it locally in your project, not globally)
1. `npx jetify` or `npx jetify -w=1` (to specify the number of parallel workers)
1. `npx react-native run-android` (your app should correctly compile and work)
1. Call `npx jetify` run in the postinstall target of your package.json (Any time your dependencies update you have to jetify again)

Proof it works / how this is tested: <https://github.com/mikehardy/rn-androidx-demo>. You can clone that repo, run the script, and see it works. Please feel to make PRs to that repo, especially in App.js or in the dependencies included, if you would like to demonstrate success or failure for a specific module.

*Inspiration:* this jetify command was based on [an idea](https://gist.github.com/janicduplessis/df9b5e3c2b2e23bbae713255bdb99f3c) from @janicduplessis - thank you Janic!

### To reverse-jetify / convert node_modules dependencies to Support Libraries

Maybe you are in the position where you must not migrate to AndroidX yet. But your libraries have started to migrate and they ship AndroidX native Java code.

You can convert them back with reverse-jetify mode

Follow the instructions from above to convert to AndroidX, **but add the `-r` flag to the `npx jetify` call.**

If a library ships only as a jar/aar/zip file, you will have to use jetifier-standalone to convert that as well, but you can delay the AndroidX migration indefinitely with this style.

## Usage for jar/zip/aar files

You may be a library maintainer, wanting to ship an AAR of your support code converted to AndroidX, or maybe you ship an AAR normally and you want to continue to support your non-AndroidX users even after you convert your library to AndroidX?

As part of your build process you can use this tool like so:

1. `npm install jetifier` (or maybe `npm install -g jetifier` to make it globally available)
1. `npx jetifier-standalone <your arguments here>` (use `npx jetifier-standalone -h` for help)

I have not altered the jetifier-standalone distribution in any way.

Other than the npm-specific instructions, consult [the official jetifier documentation](https://developer.android.com/studio/command-line/jetifier)

**Note that this is implemented for you if you [integrate the bob build tool](https://github.com/react-native-community/bob/blob/master/README.md#L44)**

## Troubleshooting

Unfortunately jetifier can't solve all your problems. Here are some reasons it could fail:

1. You have a dependency that packages things in violation of Android packaging rules, like including an extra AndroidManifest.xml or similar: <https://github.com/acaziasoftcom/react-native-bottomsheet/pull/23> - this will lead to dex merger issues about duplicate entries. Open pull requests with the libraries that do this.
1. You have a dependency that does not allow overrides of compileSdk, so you can't set the compileSdk to 28 as AndroidX requires: <https://github.com/razorpay/react-native-razorpay/pull/201>. This can lead to errors in resource merger where styles reference unknown attributes. Open pull requests with the libraries that do this

So far there has not been a case of `npx jetify` failing that wasn't based in an error in a library, so if you have a problem please examine the error and the dependency very closely and help the libraries with fixes.

### Getting it working in Windows

Jetify is a bash script so you need an updated WSL to make it work with bash, find and sed installed.

First install jetifier from a Windows command prompt:

    npm i --save-dev jetifier

Then from WSL, you can run it using:

    npx jetify

...or if that doesn't work

    ./bin/node_modules/jetify

### Performance Notes (how to set workers parameter)

In testing, it appeared that performance improved up to the number of virtual cores on a system, and then was flat but did not degrade after that no matter how many extra workers there were. So the default of 20 should result in maximum performance on even powerful systems, but smaller CI virtual machines should be fine as well. 

You will want a bash version of 5 or higher for best results. bash version 4.x ships as the default on macOS up to at least 10.14.5. Replacing the system bash with a modern bash from (for example) brew is your responsibility if you want higher performance.

Your mileage may vary.

## Contributing

Please feel free to pull requests or log issues, especially to update versions if I somehow fail to notice an update.

I have tried to make it easy for contributors to propose changes, by providing a test suite so you can safely make a change and see if it works.

I have [continuous integration enabled](https://travis-ci.com/mikehardy/jetifier) so we can prove changes work and you can make changes safely, it should pass those tests before you submit for review.

You may need to fork the test suite [rn-androidx-demo](https://github.com/mikehardy/rn-androidx-demo) if you need to add a new react-native module to test, or if you are doing something other than modifying 'jetify' (for instance if you install a python or javascript version - you'll need to [copy the git version of your new script-under-test in rn-androidx-demo/make-demo.sh](https://github.com/mikehardy/rn-androidx-demo/blob/master/make-demo.sh#L76) so it is testing your changes). Then you would [alter the .travis.yml temporarily to point to your fork of rn-androidx-demo](https://github.com/mikehardy/jetifier/blob/master/.travis.yml#L38) so that your jetifier changes were working against the updated test suite. That's all pretty annoying and I will probably move the test suite so it is internal to jetifier in the future (PRs to do that welcome...)

Thanks!
