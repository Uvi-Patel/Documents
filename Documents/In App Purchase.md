
# In App Purchase



## Installation

```bash
  pod 'SwiftyStoreKit'
```
    
## Add File

InAppPurchaseHelper.swift

```bash
import Foundation
import SwiftyStoreKit

let receiptData = SwiftyStoreKit.localReceiptData
let receiptString = receiptData!.base64EncodedString(options: [])

// MARK:- ------------------Retrieve products info------------------
func getProductDescription(product_name: String,completion: @escaping(_ result:RetrieveResults?) -> Void){
    SwiftyStoreKit.retrieveProductsInfo([product_name]) { result in
        completion(result)
    }
}

// MARK:- -----------------Purchase a product Non-Automatic given a product id-----------------
func purchaseProductNonAutomatic(product_name:String, complation: @escaping(_ result: PurchaseResult?) -> Void){
    SwiftyStoreKit.purchaseProduct(product_name, quantity: 1, atomically: true) { result in
        complation(result)
    }
}

// MARK:- -----------------Restore previous purchases Non-Atomic-----------------
func restoreProductAutomatic(complation: @escaping(_ result: RestoreResults) -> Void){
    SwiftyStoreKit.restorePurchases(atomically: true) { results in
        complation(results)
    }
}
```
RemoveAdsVC.swift

```bash
 @IBAction func clickOnRestore(_ sender: Any) {
        self.restorePurchase()
    }
    
    @IBAction func clickOnRemoveAds(_ sender: Any) {
        self.removeAds()
    }

    func removeAds(){
        self.startLoad()
       // productId = bundleID
        purchaseProductNonAutomatic(product_name: productId) { (result) in

            if case .success(let purchase) = result {
                let downloads = purchase.transaction.downloads
                if !downloads.isEmpty {
                    SwiftyStoreKit.start(downloads)
                }

                if purchase.needsFinishTransaction {
                    SwiftyStoreKit.finishTransaction(purchase.transaction)
                }
            }
            if let alert = self.alertForPurchaseResult(result!) {
                self.showAlert(alert)
            }
        }
    }
    
    func restorePurchase(){
        self.startLoad()
                
        restoreProductAutomatic { (results) in

            for purchase in results.restoredPurchases {
                let downloads = purchase.transaction.downloads
                if !downloads.isEmpty {
                    SwiftyStoreKit.start(downloads)

                } else if purchase.needsFinishTransaction {
                    // Deliver content from server, then:
                    SwiftyStoreKit.finishTransaction(purchase.transaction)
                }
            }
            self.showAlert(self.alertForRestorePurchases(results))
        }
    }
    extension RemoveAdsVC{
    func alertWithTitle(_ title: String, message: String) -> UIAlertController {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .cancel, handler: nil))
        return alert
    }

    func showAlert(_ alert: UIAlertController) {
        guard self.presentedViewController != nil else {
            self.present(alert, animated: true, completion: nil)
            return
        }
    }

    func alertForProductRetrievalInfo(_ result: RetrieveResults) -> UIAlertController {
        if let product = result.retrievedProducts.first {
            let priceString = product.localizedPrice!
            return alertWithTitle(product.localizedTitle, message: "\(product.localizedDescription) - \(priceString)")
        } else if let invalidProductId = result.invalidProductIDs.first {
            return alertWithTitle("Could not retrieve product info", message: "Invalid product identifier: \(invalidProductId)")
        } else {
            let errorString = result.error?.localizedDescription ?? "Unknown error. Please contact support"
            return alertWithTitle("Could not retrieve product info", message: errorString)
        }
    }

    // swiftlint:disable cyclomatic_complexity
    func alertForPurchaseResult(_ result: PurchaseResult) -> UIAlertController? {
        switch result {
        case .success(let purchase):
            UserDefaults.standard.set(true, forKey: "isPurchased")
            UserDefaults.standard.synchronize()
            NotificationCenter.default.post(name: .isPurchased, object: nil)
            print("Purchase Success: \(purchase.productId)")
            self.stopLoad()
            self.dismiss(animated: true, completion: nil)
            return nil
        case .error(let error):
            self.stopLoad()
            print("Purchase Failed: \(error)")
            switch error.code {
            case .unknown: return alertWithTitle("Purchase failed", message: error.localizedDescription)
            case .clientInvalid: // client is not allowed to issue the request, etc.
                return alertWithTitle("Purchase failed", message: "Not allowed to make the payment")
            case .paymentCancelled: // user cancelled the request, etc.
                return nil
            case .paymentInvalid: // purchase identifier was invalid, etc.
                return alertWithTitle("Purchase failed", message: "The purchase identifier was invalid")
            case .paymentNotAllowed: // this device is not allowed to make the payment
                return alertWithTitle("Purchase failed", message: "The device is not allowed to make the payment")
            case .storeProductNotAvailable: // Product is not available in the current storefront
                return alertWithTitle("Purchase failed", message: "The product is not available in the current storefront")
            case .cloudServicePermissionDenied: // user has not allowed access to cloud service information
                return alertWithTitle("Purchase failed", message: "Access to cloud service information is not allowed")
            case .cloudServiceNetworkConnectionFailed: // the device could not connect to the nework
                return alertWithTitle("Purchase failed", message: "Could not connect to the network")
            case .cloudServiceRevoked: // user has revoked permission to use this cloud service
                return alertWithTitle("Purchase failed", message: "Cloud service was revoked")
            default:
                return alertWithTitle("Purchase failed", message: (error as NSError).localizedDescription)
            }
        }
    }

    func alertForRestorePurchases(_ results: RestoreResults) -> UIAlertController {

        if results.restoreFailedPurchases.count > 0 {
            print("Restore Failed: \(results.restoreFailedPurchases)")
            self.stopLoad()
            return alertWithTitle("Restore failed", message: "Unknown error. Please contact support")
        } else if results.restoredPurchases.count > 0 {
            print("Restore Success: \(results.restoredPurchases)")
            UserDefaults.standard.set(true, forKey: "isPurchased")
            UserDefaults.standard.synchronize()
            NotificationCenter.default.post(name: .isPurchased, object: nil)
            self.stopLoad()
            self.dismiss(animated: true, completion: nil)
            return alertWithTitle("Purchases Restored", message: "All purchases have been restored")
        } else {
            print("Nothing to Restore")
            self.stopLoad()
            return alertWithTitle("Nothing to restore", message: "No previous purchases were found")
        }
    }

    func alertForVerifyPurchase(_ result: VerifyPurchaseResult, productId: String) -> UIAlertController {

        switch result {
        case .purchased:
            print("\(productId) is purchased")
            return alertWithTitle("Product is purchased", message: "Product will not expire")
        case .notPurchased:
            print("\(productId) has never been purchased")
            return alertWithTitle("Not purchased", message: "This product has never been purchased")
        }
    }
}

```
create globle function and its call to appdelegate.swift file
```bash
func swiftyStoreKit(){
    SwiftyStoreKit.completeTransactions(atomically: true) { purchases in
        for purchase in purchases {
            switch purchase.transaction.transactionState {
            case .purchased, .restored:
                if purchase.needsFinishTransaction {
                    // Deliver content from server, then:
                    SwiftyStoreKit.finishTransaction(purchase.transaction)
                }
                // Unlock content
            case .failed, .purchasing, .deferred:
                break // do nothing
            }
        }
    }
}
```