```swift

//
//  DynamicInAppNotificationApp.swift
//  DynamicInAppNotification
//
//  Created by paige shin on 2022/10/18.
//

import SwiftUI
import UserNotifications

// MARK: CREATING APNS FILES
// Simple Create a new filew ith format: .apns
// And Paste the Following Code
/*
 {
    "aps": {
        "alert": {
            "title": "Push Notifiction's",
            "body": "In App Notification Using Dynamic Island!!!"
        },
        "badge": 0
    },
    "Simulator Target Bundle": "com.paigesoftware.DynamicInAppNotification"
 }
 */
@main
struct DynamicInAppNotificationApp: App {
    
    // MARK: Linking App Delegate
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
    
    // MARK: All Notifications
    @State var notifications: [NotificationValue] = []
    var body: some Scene {
        WindowGroup {
            ContentView()
                .frame(maxWidth: .infinity, maxHeight: .infinity)
                .overlay(alignment: .top) {
                    GeometryReader { proxy in
                        let size = proxy.size
                        
                        ForEach(notifications) { notification in
                            NotificationPreview(size: size, value: notification, notifications: $notifications)
                                .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)
                        }
                        
                    }
                    .ignoresSafeArea()
                }
                .onReceive(NotificationCenter.default.publisher(for: Notification.Name("NOTIFY"))) { output in
                    if let content = output.userInfo?["content"] as? UNNotificationContent {
                        // MARK: Creating New Notifcation
                        let newNotification = NotificationValue(content: content)
                        notifications.append(newNotification)
                    }
                }
        }
    }
}

// MARK: Creating Custom Notification View
// Which will be looking like it's extracting from dynamic island
struct NotificationPreview: View {
    
    let lonelyIslandWidth: CGFloat = 126
    let lonelyIslandHeight: CGFloat = 36.33
    let lonelyIslandTopOffset: CGFloat = 11
    
    var size: CGSize
    var value: NotificationValue
    @Binding var notifications: [NotificationValue]
    
    // MARK: FOR DEMO PURPOSE
    @State var show: Bool = false
    
    var body: some View {
        HStack {
            // MARK: UI
            // NOTE: App Icon File Can Be Accessed with this String "AppIcon60x60"
            if let image = UIImage(named: "AppIcon60x60") {
                Image(uiImage: image)
                    .resizable()
                    .aspectRatio(contentMode: .fit)
                    .frame(width: 60, height: 60)
                    .clipShape(RoundedRectangle(cornerRadius: 15, style: .continuous))
            } else {
                Rectangle()
                    .frame(width: 60, height: 60)
            }
            
            VStack(alignment: .leading, spacing: 8) {
                Text(value.content.title)
                    .fontWeight(.semibold)
                    .foregroundColor(.white)
                Text(value.content.body)
                    .font(.caption)
                    .foregroundColor(.gray)
            }
            .lineLimit(1)
            .frame(maxWidth: .infinity, alignment: .leading)
            
        }
        .frame(maxWidth: .infinity, alignment: .leading)
        .padding(10)
        .padding(.horizontal, 12)
        // MARK: IF YOUR TEXT GOES MORE THAN ONE LINE
        // Then it's Recommended to give padding as the height of the Dynamic Island
        .padding(.vertical, 18)
        // MARK: ADDING SOME BLUR
        .blur(radius: value.showNotification ? 0 : 30)
        .opacity(value.showNotification ? 1 : 0)
        .scaleEffect(value.showNotification ? 1 : 0, anchor: .top)
        
        // It's Not Matching the Curve
        // So Giving padding as the same amount of top offset
        // 11 * 2 => 22
        .frame(width: value.showNotification ? size.width - 22 : lonelyIslandWidth, height: value.showNotification ? nil : lonelyIslandHeight)
        .background(
            // RADIUS = 126/2 => 63
            RoundedRectangle(cornerRadius: value.showNotification ? 50 : 63, style: .continuous)
                .fill(.black)
        )
        .clipped()
        .offset(y: lonelyIslandTopOffset)
        .animation(.interactiveSpring(response: 0.6, dampingFraction: 0.7, blendDuration: 0.7), value: value.showNotification)
        // MARK: AUTO CLOSE AFTER SOME TIME
        // THEN REMOVING NOTIFICATION FROM THE ARRAY
        .onChange(of: value.showNotification, perform: { newValue in
            if newValue && notifications.indices.contains(index) {
                // YOUR CUSTOM VALUE GOES HERE
                
                // MARK: ADDING MULTIPLE NOTIFICATIONS AS OVERLAY
                DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
                    
                    if notifications.indices.contains(index + 1) {
                        notifications[index + 1].showNotification = true
                    }
                    
                    // 1.5 + 1.3 = 2.8
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1.3) {
                        notifications[index].showNotification = false
                        
                        // MARK: SAFE CHECK GOES HERE
                        // BEFORE REMOVING ITEM FROM THE ARRAY
                        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                            if notifications.indices.contains(index + 1) {
                                notifications[index + 1].showNotification = true
                            }
                        }
                        
                        // OUR MAX ANIMATION TIMING IS 0.7
                        // SO AFTER THAT REMOVING NOTIFICATION FROM THE ARRAY
                        DispatchQueue.main.asyncAfter(deadline: .now() + 0.7) {
                            notifications.remove(at: index)
                        }
                    }
                }
            
            }
        })
        .onAppear {
            // MARK: Animation When A New Notifcation is Added
            if index == 0 {
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                    notifications[index].showNotification = true
                }
            }

            
        }
    }
    
    // MARK: Index
    var index: Int {
        return notifications.firstIndex { CValue in
            CValue.id == value.id
        } ?? 0
    }
    
}

// MARK: APP DELEGET TO LISTEN FOR IN APP NOTIFICATION
class AppDelegate: NSObject, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        UNUserNotificationCenter.current().delegate = self
        return true
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification) async -> UNNotificationPresentationOptions {
        if UIApplication.shared.haveDynamicIsland {
            // MARK: DO ANIMATION
            // MARK: Observing Notifications s
            NotificationCenter.default.post(name: NSNotification.Name("NOTIFY"), object: nil, userInfo: ["content": notification.request.content])
            return [.sound]
        } else {
            // MARK: NORMAL NOTIFICATION
            return [.sound, .banner]
        }
    }
    
}

extension UIApplication {
    
    var haveDynamicIsland: Bool {
        return self.deviceName == "iPhone 14 Pro" || deviceName == "iPhone 14 Pro Max"
    }
    
    var deviceName: String {
        return UIDevice.current.name
    }
    
}

```
# notification_dynamic_island
