
# Alamofilre API calling



## Installation

```bash
  pod 'Alamofire'
  // if 'Alamofire' not workd also try to second pod
  pod 'AlamofireObjectMapper', '~> 5.2'
```

## Add Method (GET,POST)
```bash
//GET
  Alamofire.request("https://jsonplaceholder.typicode.com/posts/1").responseJSON { (response) -> Void in
      if let JSON = response.result.value{
         print(JSON)
      }
  }

//POST
  var urlRequest = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/albums")!)
            urlRequest.httpMethod = "POST"
            let dataDictionary = ["userId": "11",
                                  "title": "welcome"]
  Alamofire.request("https://jsonplaceholder.typicode.com/albums", method: .post, parameters: dataDictionary, encoding: URLEncoding.httpBody)
        .responseJSON { response in
            print(response)
        }

//POST Image
  let image = UIImage.init(named: "whatsapp")
  let imgData = image!.jpegData(compressionQuality: 0.2)!

  let parameters = ["title": "whatsapp","url": "whatsapp"] //Optional for extra parameter

     Alamofire.upload(multipartFormData: { multipartFormData in
                multipartFormData.append(imgData, withName: "fileset",fileName: "file.jpg", mimeType: "image/jpg")
                for (key, value) in parameters {
                        multipartFormData.append(value.data(using: String.Encoding.utf8)!, withName: key)
                    } //Optional for extra parameters
            },
        to:"https://jsonplaceholder.typicode.com/photos")
        { (result) in
            switch result {
            case .success(let upload, _, _):

                upload.uploadProgress(closure: { (progress) in
                    print("Upload Progress: \(progress.fractionCompleted)")
                })

                upload.responseJSON { response in
                     print(response.result.value)
                }

            case .failure(let encodingError):
                print(encodingError)
          }
      }

  //POST Video
  Alamofire.upload( multipartFormData: { multipartFormData in
            multipartFormData.append(videoUrl, withName: "video", fileName: "video.mp4", mimeType: "video/mp4")

        }, to: url, encodingCompletion: { encodingResult in
            switch encodingResult {
            case .success(let upload, _, _):
                upload.responseJSON { response in
                    if let JSON = response.result.value as? NSDictionary {
                        completion(true)
                    } else {
                        completion(false)
                        print(response)
                    }
                }
            case .failure(let encodingError):
                print(encodingError)
                completion(false)
          }
    })

    Note : if any error in this code so its swift version issue
```
                
            