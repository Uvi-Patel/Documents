
# OneSignal Notification
Install Pod

```bash
  pod 'OneSignal', '>= 2.11.2', '< 3.0'
```
Appdelegate.swift
```bash
import OneSignal
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    var onesignal_token:String = ""
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
  if #available(iOS 10.0, *) {
          UNUserNotificationCenter.current().delegate = self
          let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
          UNUserNotificationCenter.current().requestAuthorization(
            options: authOptions,
            completionHandler: {_, _ in })
        } else {
          let settings: UIUserNotificationSettings = UIUserNotificationSettings(types: [.alert, .badge, .sound], categories: nil)
          application.registerUserNotificationSettings(settings)
        }

        application.registerForRemoteNotifications()
        
        //MARK:- OneSignal
        let onesignalInitSettings = [kOSSettingsKeyAutoPrompt: false, kOSSettingsKeyInAppLaunchURL: false]
        OneSignal.initWithLaunchOptions(launchOptions, appId: one_signal_app_id, handleNotificationAction: nil, settings: onesignalInitSettings)
        OneSignal.add(self as OSSubscriptionObserver)
        OneSignal.inFocusDisplayType = OSNotificationDisplayType.notification
        OneSignal.promptForPushNotifications(userResponse: { accepted in
          print("User accepted notifications: \(accepted)")
        })
        
        if let userId = OneSignal.getPermissionSubscriptionState().subscriptionStatus.userId {
            onesignal_token = userId
        }
        
        UIApplication.shared.registerForRemoteNotifications()
    }   
}
extension AppDelegate: OSSubscriptionObserver{
    func onOSSubscriptionChanged(_ stateChanges: OSSubscriptionStateChanges!) {
        if !stateChanges.from.subscribed && stateChanges.to.subscribed {
            print("Subscribed for OneSignal push notifications!")
        }
        print("SubscriptionStateChange: \n\(stateChanges)")
        
        //The player id is inside stateChanges. But be careful, this value can be nil if the user has not granted you permission to send notifications.
        if let playerId = stateChanges.to.userId {
            onesignal_token = playerId
        }
    }
}
```

