# GIFGroover

## What does GIFGroover do?

It loads decodes and plays a gif. 

## What can GIFGroover be used for.

- Use GIFGroover to load a GIF as a sequence of images (frames). 
- Use GIFGroover to play gifs on canvas.
- Use GIFGroover to control gif play speed, including backwards.
- Use GIFGroover to pause and un-pause a gif.
- Use GIFGroover to add/modify individual frames.
- Use GIFGroover to change the frame sequence.
- Use GIFGroover to access the comment/s field (if available).
- Use GIFGroover to infinitely loop, limited loop gifs.

What does GIFGroover not do?

- Does not display the GIF you must do that.
- It is not a replacement for displaying GIFs. It uses much more memory than just adding a gif as a normal image.
- It does not keep perfect time.
- It can not load cross domain gifs. To load a gif from another domain it must come with a CORS header.



## Is there a demo?

Example of GIFGroover in action.

[Creating a gif player using GIFGroover at codePen][1]

https://codepen.io/Blindman67/pen/wpbaoR?editors=0010

## Important WARNING.

GIFGroover can use a lot of memory. DO NOT de-reference the gif object created by GIFGroover unless you first call pause(). 

---

## How do I use GIFGroover

### Install

Download GIFGroover.js and and then add the GifGroover to the web page

```Javascript
<script src="GIFGroover.js"></script>
```

### To create and load a gif

```Javascript
    const myGif = GIFGroover();
    myGif.src = "gifurl.gif";
```

### To display an animated GIF.

Create a canvas

```Javascript
var myCanvas = document.createElment("canvas");   
``` 

Create and load gif, adding an onload event.

```Javascript
const myGif = GIFGroover();
myGif.src = "gifurl.gif";
myGif.onload = gifLoad;
```    
    
The load event is used to set the canvas size and start the display loop.

```Javascript
function gifLoad(event) {
    const gif = event.gif;
    const canvas = document.createElment("canvas");    
    canvas.width = gif.width; // set canvas size to match the gif.
    canvas.height = gif.height;
    const ctx = canvas.getContext("2d");  // get a rendering context
    document.body.appendChild(canvas);    // add canvas to the DOM
    requestAnimationFrame(displayGif);    // start displaying the gif.
    
    // Display loop
    function displayGif(){
        ctx.clearRect(0,0,canvas.width,canvas.height); // Clear in case the gif is transparent
        ctx.drawImage(gif.image);                      // The current frame
        requestAnimationFrame(displayGif);
    }
}    
```  

----
  
## The API

### Shuttle controls

```Javascript
myGif.pause();                    // pause gif
myGif.play();                     // continues playing from last position
myGif.seek(time);                 // move gif playback to time. Time is in seconds
var time = myGif.duration;        // get the play length of the gif in seconds
myGif.seekFrame(frameIndex);      // move gif playback to frameIndex. First frame is 0
var numberFrames = myGif.frameCount;
var time  = myGif.currentTime;    // get the current time This is time at the start of displaying the current frame
var frameNum  = myGif.currentFrame; // get the current frame index
myGif.currentTime = 2;            // same as seek, seeks to the time in seconds
myGif.currentFrame = 0;           // same as seekFrame
var playSpeed = myGif.playSpeed;  // get the current play speed.
myGif.playSpeed = ?;              // sets the current playback speed.
                                  // set to zero pauses playback. 
                                  // negative numbers play backward
                                  // 0.25 is quarter speed
                                  // 0.5 is half speed
                                  // 1 is normal speed
                                  // 2 is double speed
                                  // 4 is 4 times speed
                     
                     
myGif.paused    // true if paused
myGif.loading   // true if loading
myGif.playing   // true if playing (not paused)
```

### Loading and decoding

To start loading a gif set its src to the gif URL

```Javascript
GIFGroover.src = "gifUrl.gif"; 
```

Currently this can only be set once. Setting it again will generate an error warning that the new URL is being ignored.

GIFGroover will first load the gif file. If there are any problems it will generate an error. If the file is loaded then the decode process will start. 

Upon reading the first block GIFGroover will fire the `GIFGroover.ondecodestart` event which contains the width and height of the gif. It is unknown at this point how many frames the gif contains.

If you set `firstFrameOnly` to true GIFGroover will only decode the very first frame. This is handy if you want to quickly preview the content of a file.

```Javascript
GIFGroover.firstFrameOnly = true; // default is false
```
    
While decoding GIFGroover will fire `onprogress` event containing the fraction of file data decoded (0-1) and the number of frames decoded. At any time you may cancel the decoding by calling `GIFGroover.cancel`. If GIFGroover is working on a frame it will complete the frame before stopping. A canceled gif can still play but will only play the frames loaded.

```Javascript
GIFGroover.cancel(); // call to stop decoding
```
When decoding is complete the `onload` event is fired. It contains the number of frames decoded. The gif is now ready to play. If you have the `GIFGroover.playOnLoad` flag set to true the gif will start playing at the same time.    
    
```Javascript    
// set to false to prevent play at load.
GIFGroover.playOnLoad = false;  // default is true
```
    
## Disposing of the GIFGroover object.

GIFGroover can use a lot of memory so you must be careful when you no longer need the gif and de-reference it. GIFGroover uses an internal timeout event to update frames as it plays, this event has closure over the gif object. If you de-reference the gif while it is playing it will not be deleted, nor will you be able to access the gif to stop it. This can be a serious memory leak.

WHEN EVER YOU Dereference the gif object call pause befor you do.

```Javascript
var myGif = GIFGroover();      // create an load a gif
myGif.src = "gifFilename.gif"; 


// Now you longer need the gif
// The wrong way to delete it
// DONT replace it
myGif = GIFGroover(); // If gif playing you just lost access and can not release the memory it hold

// DONT dereference it
myGif = undefined;    // If gif playing you just lost access and can not release the memory it hold

// WITHOUT First calling pause;
myGif.pause();
myGif = GIFGroover();  // GOOD

// or    
myGif.pause();
myGif = undefined;  // GOOD
```
    
## Events

There are several events 

```Javascript
myGif.onerror;       // Called if there is an error during loading and decoding
myGif.onload;        // Called when the gif has finished loading and decoding
myGif.onprogress;    // Called as the decode process is underway    
myGif.ondecodestart; // Called after file load and the basic header info read containing image width and height
```

Each event has the same following properties in the event argument

```Javascript
gifEvent = {
    type : "eventname", // a string contains the event name
    gif : gif,          // reference to the gif
};
```

The following is a list of additional event object properties

```Javascript
gifError = {
    message : "string",  // string containing the error message
};
gifProgress = {
    progress : number,   // progress as a fraction 0 start of decode 1 done and about to fire onload
};
gifLoad = {
    frameCount : number, // number of frames loaded
};
gifDecodeStart = {
    width : number,  // width of gif in pixels
    height : number,  // height of gif in pixels
}
```

## Background and comments.

```Javascript
GIFGroover.backgroundColor; // read only background colour
```
    
A gif can hold a background colour. You can read the background color via the property `GIFGroover.backgroundColor`. The color is a CSS rgb color eg `rgb(255,255,255)` or if there is a problem the color will be black.

```Javascript
GIFGroover.comments; // array of comments found in the gif.
```
    
Gifs can also hold comments. You can get the array of comments (one for each comment block found) via `GIFGroover.comments`. If no comments were found the array will be empty.
    
## Random access frames.    

You can get random access to the gif frames via the frames property

```Javascript
GIFGroover.frames; // returns an array of all frames
```

or a single frame 

```Javascript
GIFGroover.getFrame(10); // return frame index 10 (the 11th frame)
```

A frame is a `HTMLCanvasElement`. You can access the canvas 2D context via its context property.

Example modifying a frame's content

```Javascript
var gifFrame = myGif.getFrame(10);
gifFrame.ctx.font = "18px arial";
gifFrame.ctx.fillStyle = "white";
gifFrame.ctx.fillText("F10",10,18);  //add the text F10 to the top corner of the frame
```

Changing the frame size will clear the frame, unless you save the content there is no way to get the frame image back using GIF). Changing the size will also produce unexpected results when playing the gif.

----
## Things to do.

- Add frame events that fire on selected frames.
- Add ability to add, move, remove frames.
- Add access to color tables.
- Possible WebGL render that would allow gif frames to be stored as compressed textures, or indexed pixel arrays and colour pallets. This would greatly reduce the memory footprint.
- Add webWorker to decode the gif.
- Write a web-Assembly lzwDecoder which is currently the slow point of this API.
- Fix the badly encoded gifs that incorrectly use transparent to save space.

----
## Questions you may ask.

- Is there a encoder? Yes but it is far from ready, I am but one person and have a large plate full.
- Will this work on IE11? As it is NO. If you want it to run on IE you will need to use a transpiler, like Babel as GIFGroover uses ES6 features that are not supported by IE
- Will this work on all browsers? NO. Only the big 3 Firefox, Chrome, and Edge. Though it may well work on others.
- Will you be adding legacy browser support? NO never!
- Why do some gifs have holes in them? Some gifs save space by not including pixels that match the background color. You will need to 
   .1 modify the frames by rendering the background color using ctx.globalCompositeOperation = "destination-in" 
   .2 or set the background colour to the correct value.
   There is no way to know if a gif is transparent because it is meant that way or if the transparency is a way of saving space. Of the 1000's of random gifs I have tested only two use transparency incorrectly, so a fix may well never eventuate.
   
  


  [1]: https://codepen.io/Blindman67/pen/wpbaoR?editors=0010