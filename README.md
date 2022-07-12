# Deep_Block_Image_Segmentation_API_Tutorial

![banner](https://github.com/omnis-labs-company/Deep_Block_Image_Segmentation_API_Tutorial/blob/main/thumbnail.JPG)

This is an example project to highlight the usage of our API for Deep Block to build applications using AI. It is a website which takes the camera input from your webcam and performs image segmentation to find your hair. You can use the source code as a guide to build your own application utilizing our API. 

## Running The Website

To run this website locally, first clone the repository:

```
git clone git@github.com:omnis-labs-company/Deep_Block_Image_Segmentation_API_Tutorial.git
```

To use the API you will need to provide an API key to authenticate yourself and a project key to specify which Deep Block project you want to use.
You need to place them in

```
function runInference() {
        const projectKey = "Your project ID here";
        const apiKey = "Your API key here";
        ...
}
```

To get your API key, visit https://app.deepblock.net/document and press GET NEW KEY.

For the project key, visit https://app.deepblock.net/deepblock/console and copy the Project Key entry for the project you want to use.
To use the same hair segmentation project we used, first fork the project from the project store at https://app.deepblock.net/deepblock/store/project/plilk4eckee2dsd9 and then copy the project key from the console.

Next, run

```
cd Deep_Block_Image_Segmentation_API_Tutorial
```

```
http-server
```

To install http-server, see https://www.npmjs.com/package/http-server

Once you opened the website, you can capture a picture from your connected webcam by pressing 'Take a Picture' and then run inference to find your hair by pressing 'Find hair'.

## Guide Through The API Usage

Before using a project through the API, you need to either create a project on https://app.deepblock.net/deepblock/console and train it or fork an already existing one from https://app.deepblock.net/deepblock/store/project. You can use our API to run inference on any projects in your console.

First you need to upload the images you want to run inference on through an HTTP post request:

```
  const takenPictureCanvas = document.getElementById("takenPicture");
  const dataURL = takenPictureCanvas.toDataURL("image/jpeg");
  const blob = dataURItoBlob(dataURL);
  const uploadTimestamp = Date.now().toString();
  const fileName  = `${uploadTimestamp}.jpeg`;

  const file = new File([blob], fileName);

  const uploadFormData = new FormData();
  uploadFormData.append('files', file);

  const url = `https://app.deepblock.net/storage_api/file_system/image_segmentation/projects/${projectKey}/predict-files?store=false`;

  fetch(url, {
    method: "POST",
    headers: {
      Authorization: apiKey,
      timestamp: uploadTimestamp
    },
    body: uploadFormData
  })
```

To do this, you need to create a [File](https://developer.mozilla.org/en-US/docs/Web/API/File) object with the image data and set it as the 'files' entry in a [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object which you put as the body of the request. You also need to put the projectKey of the specific project you want to use as a parameter in the URL as shown above. Your API key and a current timestamp need to be specified as headers.

If you want to run inference on only the images you upload right now, you first need to make sure that the project does not contain any previously uploaded images for inference. To do this, you can use the remove predict files API call before uploading your images:

```
fetch(`https://app.deepblock.net/storage_api/file_system/image_segmentation/projects/${projectKey}/predict-files`, {
  method: "DELETE",
  headers: {
    Authorization: apiKey,
    timestamp: Date.now()
  },
})
```

Next, you need to use the following HTTP request to start running inference on the uploaded images:

```
fetch(`https://app.deepblock.net/master_api/projects/${projectKey}/predict/${thresholdScore}`, {
  method: "POST",
  headers: {
    Authorization: apiKey,
    timestamp: Date.now()
  },
})
```

Here, thresholdScore is the level of confidence the model needs to have in its predictions. It has to be a value between 0 and 100. The higher the value, the higher the confidence, but less predictions will be made. For many applications, a value between 30 and 70 is reasonable.

To know when the inference is complete, you need to periodically check the status of the project. This can be done through this HTTP request:

```
fetch(`https://app.deepblock.net/master_api/projects/${projectKey}/project-status`, {
  method: "GET",
  headers: {
    Authorization: apiKey,
    timestamp: Date.now()
  }
})
```
If the status is equal to 0, inference is complete and you can proceed with getting the results:

```
fetch(`https://app.deepblock.net/master_api/projects/${projectKey}/result`, {
  method: "GET",
  headers: {
    Authorization: apiKey,
    timestamp: Date.now()
  }
})
```

This will return the predictions in the format of a JSON file which has the entries 'images', 'annotations' for each image and 'categories'. You can use the predictions specified in 'annotations' to complete your task! Additionally, if you need an easy way to visualize the predictions of an image, we have an API call for that as well:

```
fetch(`https://app.deepblock.net/storage_api/file_system/image_segmentation/projects/${projectKey}/visualized/base64/${imageID}`, {
  method: "GET",
  headers: {
    Authorization: apiKey,
    timestamp: Date.now()
  }
})
```

For this, you need to specify the imageID of the image you want to visualize. You can find it in the 'images' entry of the JSON file you just got. This HTTP request will return the visualized predictions as a base64 encoded image.

For more details, take a look at the documentation for each API call on https://app.deepblock.net/document.
