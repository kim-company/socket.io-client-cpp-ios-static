# Repository info
This repository should help everyone to build a staic c++ socket.io-client-cpp library for iOS and macOS. The library socket.io-client-cpp is maintained under https://github.com/socketio/socket.io-client-cpp. This library is needed for WebRTC frameworks as OWT https://github.com/open-webrtc-toolkit/owt-client-native 

# Build socket.io-client-cpp
In this manual we plan to use the library `socket.io-client-cpp` for the iOS client Framework of OWT https://github.com/open-webrtc-toolkit/owt-client-native. To run later OWT.framework and WebRTC.framework within your iOS App, we need `libsioclient.a` (or if SSL is used `libsioclient_tsl.a`). To build these files you need some time and normally a lot of patience. 

To be much faster, we also deliver a precompiled version under this link: [static-build folder](https://github.com/kim-company/socket.io-client-cpp-ios-static/tree/master/static_builds/). If your want to compile it by yourself, start with the following steps. 

## Compile socket.io-client-cpp from scratch for iOS
1. We need OpenSSL build for iOS. The manual way is a little time consuming. You can use the following script to create a fresh and clean iOS OpenSSL build: [build openssl for iOS](https://github.com/kim-company/openssl-ios-static/blob/master/build-openssl-for-ios.sh).
   * If you don't want to use the script, you can download the necessary files from here: [OpenSSL static iOS builds](https://github.com/kim-company/openssl-ios-static/tree/master/static_builds/)
   * If you want to use the above script, you got and folder called "build_openssl" and the folder we need later is the following: `build_openssl/arm64/` 
   * If you created by your own, you should have a folder that contains: 
      * `lib/libcrypto.a`
      * `lib/libssl.a`
      * `includes/(a long list of .h files)`
1. Get `socket.io-client-cpp` from GitHub and change to the commit that is recommended for OWT usage. The recommended commit is currently `b1216ee428dd7d1e72368da9b12aa43bfc487c93` (May 2021). To do this, run the following commands: 
  * run the following command: `git clone --recurse-submodules https://github.com/socketio/socket.io-client-cpp.git`
  * change to the commit we need: `cd socket.io-client-cpp && git checkout b1216ee428dd7d1e72368da9b12aa43bfc487c93`
  * Run configuration: 
    ```zsh
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS=-std=c++11 -DOPENSSL_ROOT_DIR=PATH_TO_OPENSSL -DOPENSSL_LIBRARIES=PATH_TO_OPENSSL/lib -DCMAKE_OSX_ARCHITECTURES=arm64 -DCMAKE_OSX_SYSROOT=`xcode-select -print-path`/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS`xcodebuild -showsdks | grep iphoneos | egrep "[[:digit:]]+\.[[:digit:]]+" -o | tail -1`.sdk ./
    ```
     - Note: Replace "PATH_TO_OPENSSL" with the path of your OpenSSL (example: "/Users/tester/Desktop/build_openssl/arm64")
     - The command `xcode-select -print-path` gets your path to Xcode
     - the command `xcodebuild -showsdks | grep iphoneos | egrep "[[:digit:]]+\.[[:digit:]]+" -o | tail -1` shows the latest iOS release installed by Xcode. We need this to specify the compiler. 
  * Run `make`
  * Now you have two files
     - File 1: `libsioclient_tls.a` (needed if SSL encryption is used)
     - File 2: `libsioclient.a` (needed if no security is used, not recommended)
  * Copy the file you need (libsioclient_tls.a OR libsioclient.a) to your Xcode project. Please note: this is an **arm64** build. This is usable only for real iPhone, iPad or Apple Silicon Macs. You will get compile errors if you try to run your Xcode project on the simulator. If you need a Simulator version, you have to compile a **x86_64** version.
     - **Optional** Compile a x86_64 version
        - run this cmake command:  
            ```zsh
            cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS=-std=c++11 -DOPENSSL_ROOT_DIR=PATH_TO_OPENSSL -DOPENSSL_LIBRARIES=PATH_TO_OPENSSL_X86/lib -DCMAKE_OSX_ARCHITECTURES=x86_64 ./
            ```
        - Note: Replace "PATH_TO_OPENSSL_X86" with the path of your OpenSSL x86_64 version (example: "/Users/tester/Desktop/build_openssl/x86_64")
        - This version can be used for running the library within the Simulator. 
    - **Optional**: You can also create a FAT library (containing both libraries at once). So you can run your project in Simulator and on an iOS device. After creating both socket.io-client-cpp versions (see above: arm64 and x86_64), you should have 4 files (2x libsioclient_tls.a, 2x libsioclient.a).
        - To create a FAT library for "libsioclient_tsl.a", you just have to run the following command: `lipo -static x86_64/libsioclient_tls.a arm64/libsioclient_tls.a -output fat/libsioclient_tls.a -create`
    
    ## Notes
    * Since commit `af68bf3067ab45dc6a53261284e0da9afd21b636` (7. January 2021) you **don't need** the Boost c++ library anymore. This simplyfies the build process a lot. 