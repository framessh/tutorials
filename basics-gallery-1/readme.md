# Basics Gallery 1

### About

This is a tutorial on how to build a basic gallery frame in frames.sh. The included files in the folder can be uploaded into your frame applications dataset `(Application -> Datasets -> Upload Dataset)`.

**IMPORTANT**: Please refer to the documentations at https://docs.frames.sh for the description on the commands used in this tutorial.

**Application Version Required**: `v1.0.5` (to update your version, go to your application's settings and Model tab then restart your application from the runtime tab in the settings popup)

**Installations**: The included html files in this tutorial must be imported into your frame application for this tutorial to work or you can recreate these files from the your frame's view editor. The JSON file includes the code of the frame application shown in this tutorial.

These templates can be editor in your frame application's View Editor.

![Upload the included html files in your frame application datasets](https://ny-1.frames.sh/v/38875/datasets.png)

**Running**: In your frame application's editor, click "Save", "Publish", and check your frame within the view editor's Live Preview or your chosen frame debugger i.e. the Warpcast frame debugger here: https://warpcast.com/~/developers/frames

**Your Frame URL**: Your frame URL can be found in the frame applications' settings.

![Frame application settings](https://ny-1.frames.sh/v/38875/settings.png)

### Getting started

To get started, upload the included HTML files required for your frame. And import the JSON file into your frame application's logic.

![Frame application settings](https://ny-1.frames.sh/v/38875/basic-gallery-1.png)

### Interface Mode

We need to enable our applications **interface mode**. The **interface mode** indicates that the app is public facing and can accept requests.

All the application requests are assigned into a queue and processed accordingly. We can set a request's queue ID by choosing a key in the request's payload. In this example we are using the **fid** of the user who's created the request (Pressed a button, etc) - this data will be included by the Farcaster client when a user interacts with a frame.

If a queue ID is not set, a random ID is assigned. Queue IDs are useful for sticky sessions, so the frame application can remember the previous request of the user in memory.

### Asessing GET & POST Requests

When a frame is rendered on a Farcaster application, a `GET` request is sent to the frame application. Similarly, if a user clicks a button on a frame, a `POST` is sent to the frame application. To handle the GET request, the `request` system variable is checked:

```
  //! We have 3 pages on this frame - the welcome page, page 2, and page 3
  //! We want to show the landing page (welcome page), allow the user to click Next button to change to the next page & Back to the previous
  //! Note that comments are marked with //! prefix

  //! Show the default landing frame
  condition equal_to request GET;
    hop #showWelcomePage;
    exit!;
  end_if;
```

To handle our GET requests, the application hops to the `#showWelcomePage` portion of the code using the `hop` command. Hop allows the application to jump to a portion in your code indicated by code number of tags.

If this condition is not satisfied we check the `request` if it's a POST request and process it accordingly:

```
  //! Process all POST request (Button Press)
  condition equal_to request POST;

      //! Get the button ID from untrustedData
      access buttonIndex,{$untrustedData};

      //! Return values of the last function called can be accessed using @@
      variable buttonId = @@;

      //! Get the frame state
      access state,{$untrustedData};

      //! Decode the page state (since the frame spec dictates it must be encoded as JSON string in URI form)
      decode @@;

      //! Get the page value from our state (JSON object)
      access page,@@;

      //! Store the page value into a variable
      variable pageId = @@;

      //! Determine which page we want to show based on the buttonId and page
      //! {$buttonId}{$page} indicates we're combining both values of the variables
      //! By default the landing page will not provide a state (in which our page should be stated)
      //! therefore it defaults to null and so we check the {$buttonId}{$page} against 1null (1 for button 1 and null for no state)
      //! We can check {$buttonId}{$page} against multiple values, multiple values are delimited using the | character in the switch command
      //! On the first page there is only one button - "Next"
      //! The switch command takes 3 arguments, value A and, the value we want to compare value A against, and which line of code we want the app to jump to
      //! Because the line of your code can change, keeping track of which line number in the code can be difficult therefore we use tags
      //! Tags will automatically take note of the line in which the tag is situated, in this example we are using tags
      //! We refer to tags with # in our switch and or hop arguments

      switch {$buttonId}{$pageId},11|1null,#showPage2;
      switch {$buttonId}{$pageId},12,#showWelcomePage;
      switch {$buttonId}{$pageId},22,#showPage3;
      switch {$buttonId}{$pageId},13,#showPage2;

      //! In the last page, we show button 2 as "Go back to the welcome page".
      switch {$buttonId}{$pageId},23,#showWelcomePage;
      exit!;
  end_if;
```

As noted on the comments, the `POST` request is handled like so:

1. Taking the buttonIndex (which button the user has clicked) and assigning it to a variable.
2. Taking the state variable in the `POST` request payload.
3. Taking the page variable and its value in the frame's state and assigning it to a variable.
4. By using the `switch` command we can evaluate which page we want to display.

The `switch` command is the same as the `hop` command - it allows the application to jump to a portion in the application's code, skipping lines before it. The `switch` command however evaluates a condition first before executing a hop. As noted above, the switch command takes in a value and another value to compare it against, and if these 2 values are equivalents the application jumps to the indicated portion of the code in the 3rd argument i.e. `#showPage2`,`#showWelcomePage`, and or `#showPage3`.

Finally, all the page responses to each POST request are organized like so:

**Show the Welcome Page**

```
  //! Create the tag for this line of code
  tag showWelcomePage;

  //! Get the template we want to show for this page
  //! The template command takes 2 arguments - first is the image override, and the second is the template name
  //! The image override argument is a link to an image that you want your frame to display
  //! If the image override is null, the app will take a screenshot of your template and serve it as the frame image
  //! The image override is useful especially if you want the frame to use GIFs (since screenshots cannot serve motion images).
  template null,welcome-page.html;

  //! Respond to the request with the template content
  respond @@;

  //! Exit the application since we've completed the request
  exit!;
```

**Show the second page**

```
  tag showPage2;
  template null,page-2.html;
  respond @@;
  exit!;
```

**Show the third page**

```
  tag showPage3;
  template null,page-3.html;
  respond @@;
  exit!;
```
