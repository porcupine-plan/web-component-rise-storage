# Rise Storage Web Component

## Introduction

The Rise Storage Web Component uses Google’s Storage API to retrieve the URL of a file, or the URLs of all files within a folder, from Rise Storage. If the Rise Cache application is running, it will be utilized for local storage of the files, and the URLs returned by the web component will point to Rise Cache. Otherwise, if Rise Cache is not running, the browser's cache will be utilized.

Folders and files are monitored for changes if the `refresh` attribute is set, although a minimum refresh time of 5 minutes is enforced.

### Filtering
Files can optionally be filtered by file type or content type. This feature is meant to be used when returning multiple files in a folder, and so will only take effect when a `folder` attribute is specified but not a `fileName`, or if neither a `folder` nor a `fileName` attribute are specified (in which case files in the root bucket would be returned).

To filter by file type, a `fileType` attribute should be added to the `<rise-storage>` element. `fileType` can be either `image` or `video`, in which case only those files that are images (`jpg`, `png`, `bmp`, `svg`, `gif`) or HTML5 videos (`mp4`, `ogv`, `webm`) will be returned.

Alternatively, the `contentType` attribute can be used to filter by a specific content type. Multiple content types can be specified by separating them with a space.

#### Examples
```
<rise-storage companyId="my-company-id" folder="my-folder" fileType="image">
</rise-storage>

<rise-storage companyId="my-company-id" contentType="image/png image/jpeg">
</rise-storage>
```

### Sorting
Files can optionally be sorted by filename, modified date or randomly. This feature is meant to be used when returning multiple files in a folder, and so will only take effect when a `folder` attribute is specified but not a `fileName`, or if neither a `folder` nor a `fileName` attribute are specified (in which case files in the root bucket would be returned).

To specify a sort order, a `sort` attribute should be added to the `<rise-storage>` element. `sort` can be one of `name`, `date` or `random`.

To specify a direction, the `sortDirection` attribute should be used. `sortDirection` can be either `asc` or `desc`. If the sort attribute is set to `random`, then `sortDirection` is ignored. If the `sort` attribute is `name` or `date`, but `sortDirection` is not specified, the files will be sorted in ascending order.

#### Example
```
<rise-storage companyId="my-company-id" folder="my-folder" sort="name" sortDirection="desc">
</rise-storage>
```

Rise Storage Web Component works in conjunction with [Rise Vision](http://www.risevision.com), the [digital signage management application](http://rva.risevision.com/) that runs on [Google Cloud](https://cloud.google.com).

At this time Chrome is the only browser that this project and Rise Vision supports.

## Usage
To use the Rise Storage Web Component, you should first install it using Bower:
```
bower install https://github.com/Rise-Vision/web-component-rise-storage.git
```

Next, construct your HTML page. You should include `webcomponents.js` before any code that touches the DOM, and load the web component using an HTML Import. For example:
```
<!DOCTYPE html>
<html>
  <head>
    <script src="bower_components/webcomponentsjs/webcomponents.js"></script>
    <link rel="import" href="bower_components/rise-storage/rise-storage.html">
  </head>
  <body>
    <rise-storage
      companyId="my-company-id"
      folder="my-folder"
      fileName="my-image.png"
      refresh="60"></rise-storage>

    <script>
      // Wait for 'polymer-ready'. Ensures the element is upgraded.
      window.addEventListener('polymer-ready', function(e) {
        var storage = document.querySelector('rise-storage');

        // Respond to events it fires.
        storage.addEventListener('rise-storage-response', function(e) {
          if (e.detail) {
            console.log(e.detail.url); // URL to the file.
          }
        });

        storage.go(); // Call its API methods.
      });
    </script>
  </body>
</html>
```

You can get the `companyId` by copying the value of the `cid` parameter in the URL when logged into the [Rise Vision platform](http://rva.risevision.com/):

![company-id](https://cloud.githubusercontent.com/assets/1190420/7048788/0835e2e2-dde3-11e4-9b25-8e4365541351.png)

The web component returns a JSON response for each file matching the provided criteria with the following format (one message per file):
```
{
  "url": "http://url.to.file",
  "added": "boolean indicating if the file has just been added",
  "changed": "boolean indicating if the file has changed since the previous message",
  "deleted": "boolean indicating if the file has been deleted since the previous message",
}
```

### Attributes
| Attribute              | Type                                                                            | Default          |
| ---------------------- | ------------------------------------------------------------------------------- |:----------------:|
| `companyId` (required) | `<string>` The ID of the Company.                                               | `''`             |
| `folder`               | `<string>` The folder name.                                                     | `''`             |
| `fileName`             | `<string>` The file name within the folder.                                     | `''`             |
| `fileType`             | `<string>` Either `image` or `video` (for HTML5 video types).                   | `''`             |
| `contentType`          | `<string>` A specific media type (e.g. `image/png` or `video/mp4`).             | `''`             |
| `sort`                 | `<string>` One of `name`, `date` or `random`.                                   | `''`             |
| `sortDirection`        | `<string>` Either `asc` or `desc`.                                              | `''` (or `asc` if `sort` is `name` or `date`)|
| `refresh`              | `<number>` The number of minutes before Storage will be checked for changes. The minimum refresh time is 5 minutes. | `0` (no refresh) |
| `env`                  | `<string>` Either `prod` or `test`.                                             | `prod`           |

### Properties
| Property         | Type                                              | Default |
| ---------------- | ------------------------------------------------- |:-------:|
| `url`            | `<string>` The URL target of the request.         | `''`    |
| `isCacheRunning` | `<boolean>` Whether or not Rise Cache is running. | `false` |

### Events
| Event                         | Description                                                                                   |
| ----------------------------- | --------------------------------------------------------------------------------------------- |
| `rise-storage-response`       | Fired when a response is received.                                                            |
| `rise-storage-error`          | Fired when an error is received.                                                              |
| `rise-storage-empty-folder`   | Fired when the request is for a folder that is empty.                                         |
| `rise-storage-no-folder`      | Fired when the request is for a folder that does not exist.                                   |
| `rise-storage-no-file`        | Fired when the request is for a file that does not exist.                                     |
| `rise-storage-file-throttled` | Fired when the request is for a file that has been throttled.                                 |
| `rise-storage-folder-invalid` | Fired when the request is for a folder that only contains files of the wrong file type format.|
| `rise-storage-api-error`      | Fired when the request returns a response indicating an error with the Storage API.           |
| `rise-cache-error`            | Fired when an error is received from cache.                                                   |
| `rise-cache-not-running`      | Fired when rise cache is not running.                                                         |



### Methods
| Method | Description                                    |
| ------ | ---------------------------------------------- |
| `go`   | Performs an Ajax request to the specified URL. |

## Built With
- [Polymer](https://www.polymer-project.org/)
- [Polymer core-ajax](https://www.polymer-project.org/docs/elements/core-elements.html#core-ajax)
- [npm](https://www.npmjs.org)
- [Bower](http://bower.io/)
- [Gulp](http://gulpjs.com/)
- [web-component-tester](https://github.com/Polymer/web-component-tester) for testing

## Development

### Dependencies
* [Git](http://git-scm.com/) - Git is a free and open source distributed version control system that is used to manage our source code on Github.
* [npm](https://www.npmjs.org/) & [Node.js](http://nodejs.org/) - npm is the default package manager for Node.js. npm runs through the command line and manages dependencies for an application. These dependencies are listed in the _package.json_ file.
* [Bower](http://bower.io/) - Bower is a package manager for Javascript libraries and frameworks. All third-party Javascript dependencies are listed in the _bower.json_ file.
* [Gulp](http://gulpjs.com/) - Gulp is a Javascript task runner. It lints, runs unit and E2E (end-to-end) tests, minimizes files, etc. Gulp tasks are defined in _gulpfile.js_.

### Local Development Environment Setup and Installation
To make changes to the web component, you'll first need to install the dependencies:

- [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Node.js and npm](http://blog.nodeknockout.com/post/65463770933/how-to-install-node-js-and-npm)
- [Bower](http://bower.io/#install-bower) - To install Bower, run the following command in Terminal: `npm install -g bower`. Should you encounter any errors, try running the following command instead: `sudo npm install -g bower`.
- [Gulp](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md) - To install Gulp, run the following command in Terminal: `npm install -g gulp`. Should you encounter any errors, try running the following command instead: `sudo npm install -g gulp`.

The web components can now be installed by executing the following commands in Terminal:
```
git clone https://github.com/Rise-Vision/web-component-rise-storage.git
cd web-component-rise-storage
npm install
bower install
```

### Run Locally
You can access the `demo.html` file via a local web server. On Mac, execute the following command from the root directory of the web component:
```
python -m SimpleHTTPServer
```

This starts a web server on port 8000, and you can access the Rise Storage web component by navigating to `localhost:8000/demo.html`.

### Testing
You can run the suite of tests via a local web server. On Mac, execute the following command from the root directory of the web component:
```
python -m SimpleHTTPServer
```

This starts a web server on port 8000, and you can run the tests by navigating to `http://localhost:8000/test/index.html`.

### Deployment
Once you are satisifed with your changes, deploy the `components` and `rise-storage` folders to your server. You can then use the web component by following the *Usage* instructions.

## Submitting Issues
If you encounter problems or find defects we really want to hear about them. If you could take the time to add them as issues to this Repository it would be most appreciated. When reporting issues, please use the following format where applicable:

**Reproduction Steps**

1. did this
2. then that
3. followed by this (screenshots / video captures always help)

**Expected Results**

What you expected to happen.

**Actual Results**

What actually happened. (screenshots / video captures always help)

## Contributing
All contributions are greatly appreciated and welcome! If you would first like to sound out your contribution ideas, please post your thoughts to our [community](http://community.risevision.com), otherwise submit a pull request and we will do our best to incorporate it. Please be sure to submit test cases with your code changes where appropriate.

## Resources
If you have any questions or problems, please don't hesitate to join our lively and responsive community at http://community.risevision.com.

If you are looking for user documentation on Rise Vision, please see http://www.risevision.com/help/users/

If you would like more information on developing applications for Rise Vision, please visit http://www.risevision.com/help/developers/.

**Facilitator**

[Donna Peplinskie](https://github.com/donnapep "Donna Peplinskie")
