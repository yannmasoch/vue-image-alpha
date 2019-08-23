<template>

  <!-- <div> -->
      <img :src="srcFinal" />
  <!-- </div> -->
	
</template>


<script>

export default {
  name: 'ImgAlpha',
  data () {
    return {
      msg: 'This is Image Alpha',
      srcFinal: null
    }
  },
  props: {
    src: String,
    srcAlpha: String,
    bgColor: String
  },
  created () {
    
    //------------------------------
    // Init settings
    //------------------------------
    
    // Referencing the scope for myWorker.addEventListener
    var myApp = this

    // No Alpha img specified (act as a normal image) -> stop the process for this element
    if(!this.srcAlpha) {

      // Use the src image fot the srcFinal
      this.srcFinal = this.src

      // Exit the process - the following process is not required
      return
    }

    // Normalize bgColor values fomr settings.bgColor and/or from data-bg-color IF bgColor prop is present and if bgColor prop is an HEX
    var bgColor = this.bgColor && isHex(this.bgColor) ? hexToRgb(normalizeHex(this.bgColor)) : [255, 255, 255];

    // The Single File Component format does not allow to use a worker from an extermal .js file
    // We have to use a Blob in order to include the worker within the SFC
    const myWorker =  new Worker(URL.createObjectURL(new Blob(['(' + theWorker + ')()'])));


    //------------------------------
    // Create canvas
    //------------------------------

    var canvas = document.createElement('canvas');
    var ctx = canvas.getContext('2d');


    // ------------------------------
    // Init temp images
    //------------------------------

    var tmpImgColor = new Image();
    var tmpImgAlpha = new Image();
    

    // ------------------------------
    // Imges processsing and compositing
    //------------------------------

    (tmpImgColor && tmpImgAlpha).onload = function() {

      // Init canvas size
      canvas.width = tmpImgColor.width
      canvas.height = tmpImgColor.height

      // Draw RGB image on canvas and get data
      ctx.drawImage(tmpImgColor, 0, 0);
      var imageDataRGB = ctx.getImageData(0, 0, canvas.width, canvas.height);

      // Clear canvas
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Draw Alpha image on canvas and get data
      ctx.drawImage(tmpImgAlpha, 0, 0);
      var imageDataAlpha = ctx.getImageData(0, 0, canvas.width, canvas.height);

      // Clear canvas
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      
      // Send data to the worker as an object
      myWorker.postMessage(
        {
          "dataRGB": imageDataRGB.data.buffer, 
          "dataAlpha": imageDataAlpha.data.buffer,
          "bgColor": bgColor,
          "width": canvas.width,
          "height": canvas.height
        }, [imageDataRGB.data.buffer, imageDataAlpha.data.buffer]
      );


      // Receive data back from the worker and build the final image
      myWorker.addEventListener('message', function(e) {

        // Draw the returned image into the canvas
        ctx.putImageData(e.data, 0, 0)
        
        // Convert the canvas in Base64 image in order to use it in a regular <img src="... />
        myApp.srcFinal = canvas.toDataURL()
        
      }, false);

    }

    // Init tmp images SRCs - must be done after the .onload 
    tmpImgColor.src = this.src;
    tmpImgAlpha.src = this.srcAlpha;

  },
  methods: {

  }

}

function theWorker() {

  function reverseBlending(color, alpha, bgColor) {
    
    // Normalize alpha 0-255 -> 0.0-1.0
    alpha = alpha / 255;
    
    // Color Alpha Blending
    var finalColor = (color / alpha) - ((bgColor * (1.0 - alpha)) / alpha);           

    return finalColor;

  }

  self.addEventListener('message', function(e) {

    // Init Uint8 Arrays
    var uInt8RGB = new Uint8ClampedArray(e.data.dataRGB)
    var uInt8Alpha = new Uint8ClampedArray(e.data.dataAlpha)

    // Init bg RGB colors - need to put e.data.bgColor in another var in order to increase the following process speed
    var bgColor = e.data.bgColor

    for (var i = 0; i < uInt8RGB.byteLength; i += 4) {
      uInt8RGB[i] = reverseBlending(uInt8RGB[i], uInt8Alpha[i], bgColor[0]);
      uInt8RGB[i+1] = reverseBlending(uInt8RGB[i+1], uInt8Alpha[i], bgColor[1]);
      uInt8RGB[i+2] = reverseBlending(uInt8RGB[i+2], uInt8Alpha[i], bgColor[2]);
      uInt8RGB[i+3] = uInt8Alpha[i];
    }

    // Create an imageData from the uInt8RGB
    let imgData = new ImageData( 
      uInt8RGB,
      e.data.width,
      e.data.height
    );

    // Return the final image
    self.postMessage(imgData);

    // Close the worker
    self.close()
    
  }, false);

}


// Test if string a correct hexstring
function isHex(hex) {

    return /^#?[0-9A-F]{6}$/i.test(hex);
    /*
        ^ match beginning
        # a hash
        ? optional char
        [a-f0-9] any letter from a-f and 0-9
        {6} the previous group appears exactly 6 times
        $ match end
        i ignore case
    */

}

// Remove '#'
function normalizeHex(hex) {
    return hex.replace('#', '');
}


// Convert an HEX string to a RGB array
function hexToRgb(hex) {
    var arrBuff = new ArrayBuffer(4);
    var vw = new DataView(arrBuff);
    vw.setUint32(0, parseInt(hex, 16), false);
    var arrByte = new Uint8Array(arrBuff);

    return [ arrByte[1],  arrByte[2], arrByte[3] ];
}

</script>


<style scoped>

</style>
