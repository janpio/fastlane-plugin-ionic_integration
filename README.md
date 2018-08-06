# cordova_screenshots plugin

WORK IN PROGRESS!

[![fastlane Plugin Badge](https://rawcdn.githack.com/fastlane/fastlane/master/fastlane/assets/plugin-badge.svg)](https://rubygems.org/gems/fastlane-plugin-cordova_screenshots)

## Getting Started

This project is a [_fastlane_](https://github.com/fastlane/fastlane) plugin. To get started with `fastlane-plugin-cordova_screenshots`, add it to your project by running:

```bash
fastlane add_plugin cordova_screenshots
```

## About cordova_screenshots

This plugin enables you do create automated screenshots of your Cordova (or Ionic) project for iOS <!--  TODO , both for iOS or Android, --> with _fastlane_ using the normal `capture_ios_screenshots` (`snapshot`) <!-- TODO and `capture_android_screenshots` (`screengrab`) --> actions.

Usually this is a challenge, as both require you to modify your native projects and add some files. As Cordova projects are generated for you and not tracked by version control, those changes would get lost frequently.

By keeping your test files in your `fastlane` folder and offering a simple action to "patch" your native projects each time before running the screenshot creation process, this plugin offers a way around that.

## Actions

### `ionic_ios_config_snapshot`

This action get's you started with a sample UI Test configuration (and saves it to `fastlane/ionic/config/ios/ui-tests`). The UI Unit Tests are linked into any existing XCode project generated by Ionic.

For example: `fastlane run ionic_ios_config_snapshot ionic_scheme_name:"ionic-screen-shots"`

*You can also specify which Xcode project to use and minimum iOS version using `ionic_ios_xcode_path` and `ionic_min_target_ios` options respectively. The Xcode project will be autodetected as the project (`.xcodeproj`) associated with the workspace (`.xcworkspace`) in `platforms/ios` folder generated by Ionic/Cordova.*

The command above will create a folder `fastlane/ionic/config/ios/ui-tests/ionic-screen-shots`

`ionic_ios_config_snapshot` also executes the `ionic_config_snapshot` action to retrofit this unit test configuration into any existing Xcode Project.

When this is done. The sample will be linked into your generated project. In the above example, a UI Test scheme will be created in `fastlane/ionic/config/ios/ui-tests/ionic-screen-shots`. The files in this folder are linked absolutely into Xcode, that is Xcode refers directly to these files (they are not copied over to Xcode).

#### Writing test code

When you open up Xcode and open the file `ionic-screen-shots/ui-snapshots.swift` you will see something like this:

```swift
    func testSnapshots() {

        //
        // Place your own tests here. This is a starter example to get you going..
        //
        snapshot("app-launch")
        
        // XCUIApplication().buttons["Your Button Name"].tap()
        
        // snapshot("after-button-pressed")
                
    }
```

In the Xcode UI, select the scheme 'ionic-screen-shots' and click into the method 'testSnapshots()' (after the first snapshot):

<img src="https://cloud.githubusercontent.com/assets/12442390/25378856/2d2d09c8-29a3-11e7-8e1b-e7d4dc3d4543.png" width="490" height="60" alt="Select UI Scheme"/>

You can now click on the 'Record UI Test' (Red Circle Icon):

<img src="https://cloud.githubusercontent.com/assets/12442390/25378879/38282d3a-29a3-11e7-8a5a-50845422f819.png" width="490" height="425" alt="Record UI Test"/>

This will open the simulator and you can click around in your application. XCode will record, each interaction within the `testSnapshots()` method.

When you are done, you can save everything and it will save those interactions into the `fastlane/ionic/config/ios/ui-tests/ui-snapshots.swift`.

You can now add fastlane `snapshot("decription")` where you like.

This whole operation only needs to be done once (or if you want to add more screenshots later).

The UI Test files can be added to your source control and they will be retrofitted into any future generated Ionic projects by the `ionic_ios_snapshot` action.

### `ionic_ios_snapshot`

Scans the `fastlane/ionic/config/ios/ui-tests` folder for sub folders that represent UI Test Schemes (each folder is a scheme). It retrofits each UI Test scheme into the XCode projects generated by Ionic. This should be executed before each time you perform [`fastlane snapshot`](https://docs.fastlane.tools/actions/snapshot/) or use the [`capture_ios_screenshots` action](https://docs.fastlane.tools/actions/capture_ios_screenshots/).

#### Retrofitting UI Test Scheme via Fastlane

After you have created your first UI Test Scheme using `ionic_ios_config_snapshot`, you will now need to ensure the unit tests are retrofitted into the Xcode project generated by Ionic/Cordova.

The `ionic_ios_snapshot` action can be executed before any other lane in your `Fastfile` by placing it in a [before_all](https://github.com/fastlane/fastlane/blob/master/fastlane/docs/Advanced.md) block.

The parameters for `ionic_ios_snapshot` action will mostly be autodetected. It attempts to pick the Xcode project by looking in `platforms/ios` folder. The `team_id` parameter is automatically set if it is specified in your fastlane `Appfile`. The `bundle_id` is set automatically if the `package_name` parameter is set in your fastlane `Appfile`. All these parameters can be overridden by specifying them in the call to the action. To see what has been automatically picked up for your configuration, use `fastlane action ionic_ios_snapshot`

`ionic_ios_snapshot` lists all the subfolders in the folder `fastlane/ionic/config/ios/ui-tests*`. It then creates a UI Test Target and Scheme based on the name of the subfolder, in the example above, this would be a single target/scheme called `ionic-snap-shots`. It effectively 'retrofits' the UI tests into the iOS Xcode project for you.

After `ionic_ios_snapshot` is executed, you can now specify any of the schemes you generated to be used in fastlane [snapshot](https://github.com/fastlane/fastlane/tree/master/snapshot) action, using the 'scheme' parameter.

Your fully automated Fastlane file can now look like this:

```ruby
platform :ios do

  before_all do
    #
    # This will retrofit any existing schemes in fastlane/ionic/config/ios/ui-tests/
    #
    ionic_ios_snapshot(
    	team_id: "[YOUR TEAM ID]"
    	bundle_id: "[YOUR APP BUNDLE ID]"
    )
  end

  lane :release do
  	# This will run the retrofitted UI Tests for you and create snapshots... :-)
  	snapshot(
      output_simulator_logs: true,
      reinstall_app: false,
      erase_simulator: true,
      scheme: "ionic-screen-shots" # The name of a scheme to use, one that you generated with ionic_ios_config_snapshot
    )
  end
end
```

You can now use this to fully automate the build process between ionic and fastlane, including snapshots.

There is also no limit to the number of different schemes you can create and use, so you could have one scheme for one lane and a whole different scheme for another lane.

## Example

Check out the [example `Fastfile`](fastlane/Fastfile) to see how to use this plugin. Try it by cloning the repo, running `fastlane install_plugins` and `bundle exec fastlane test`.

## Acknowledgement

The iOS part of this plugin is fully based on knocknarea's excellent [`fastlane-plugin-ionic_integration`](https://github.com/knocknarea/fastlane-plugin-ionic_integration). I forked and simplified his existing plugin and added the Android implementation.

## Run tests for this plugin

To run both the tests, and code style validation, run

```
rake
```

To automatically fix many of the styling issues, use
```
rubocop -a
```

## Issues and Feedback

For any other issues and feedback about this plugin, please submit it to this repository.

## Troubleshooting

If you have trouble using plugins, check out the [Plugins Troubleshooting](https://docs.fastlane.tools/plugins/plugins-troubleshooting/) guide.

## Using _fastlane_ Plugins

For more information about how the `fastlane` plugin system works, check out the [Plugins documentation](https://docs.fastlane.tools/plugins/create-plugin/).

## About _fastlane_

_fastlane_ is the easiest way to automate beta deployments and releases for your iOS and Android apps. To learn more, check out [fastlane.tools](https://fastlane.tools).

