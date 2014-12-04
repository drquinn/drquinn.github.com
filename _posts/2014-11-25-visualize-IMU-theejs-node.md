---
layout: post
title: Visualize IMU Data with Three.js and Node.js
---

[Code is posted here on GitHub.](https://github.com/drquinn/imu-visualize)

The Adafruit 10DOF is an inertial measurement unit (IMU).  Adafruit has an [excellent guide](https://learn.adafruit.com/adafruit-10-dof-imu-breakout-lsm303-l3gd20-bmp180) for getting the device running.

This particular IMU uses 3 different sensors: L3DG20H gyroscope + LSM303DLHC accelerometer compass + BMP180 barometric/temperature sensor.

My goal was to view movement in the IMU on a web page. The end result looks like this:

![IMU Visualized](/assets/IMUvis/imu-vis.gif "IMU Visualized")

 This was achieved using an Arduino to capture data from the IMU. I loaded code provided by Adafruit onto the Arduino to get AHRS data.  Then used [Node.js](http://nodejs.org/) to capture the serial port stream from the Arduino and pass that data client side using [socket.io](http://socket.io/). Finally once on the client, using the [three.js library](http://threejs.org/) to render a cube.

First, follow the [Adafruit guide](https://learn.adafruit.com/adafruit-10-dof-imu-breakout-lsm303-l3gd20-bmp180) to load all helper libaries for the IMU. Then load the [AHRS code from Adafruit](https://learn.adafruit.com/ahrs-for-adafruits-9-dof-10-dof-breakout/using-adafruit-ahrs) onto the Arduino.

Next, install the following node modules onto the machine you plan to use as your sever with npm:

* [socket.io](https://www.npmjs.org/package/socket.io)
* [serialport](https://www.npmjs.org/package/serialport/)

Node will run the following app.js file:

	var app = require('http').createServer(handler);
	var url= require('url');
	var fs = require('fs');
	var os = require('os');
	var io = require('socket.io').listen(app);
	var serialport = require("serialport");
	var SP = serialport.SerialPort;
	var serialPort = new SP("/dev/ttyUSB0",
		{
			baudrate: 115200,
			parser: serialport.parsers.readline("\n")
		}, false);
	
	app.listen(5000);
	
	/* SERIAL WORK */
	
	serialPort.open(function (error) {
	  if ( error ) {
	    console.log('failed to open: '+error);
	  } else {
	    console.log('open');
	    serialPort.on('data', function(data) {
	      console.log('data received: ' + data);
	      io.sockets.emit('serial_update', data);
	    });
	    //serialPort.write("ls\n", function(err, results) {
	    //  console.log('err ' + err);
	    //  console.log('results ' + results);
	    //});
	  }
	});
	
	
	// Http handler function
	function handler (req, res) {
	    
	    // Using URL to parse the requested URL
	    var path = url.parse(req.url).pathname;
	    
	    // Managing the root route
	    if (path == '/') {
	        index = fs.readFile(__dirname+'/three.html', 
	            function(error,data) {
	                
	                if (error) {
	                    res.writeHead(500);
	                    return res.end("Error: unable to load three.html");
	                }
	                
	                res.writeHead(200,{'Content-Type': 'text/html'});
	                res.end(data);
	            });
	
	    // Managing the route for the javascript files
	    } else if( /\.(js)$/.test(path) ) {
	        index = fs.readFile(__dirname+path, 
	            function(error,data) {
	                
	                if (error) {
	                    res.writeHead(500);
	                    return res.end("Error: unable to load " + path);
	                }
	                
	                res.writeHead(200,{'Content-Type': 'text/plain'});
	                res.end(data);
	            });
	    } else {
	        res.writeHead(404);
	        res.end("Error: 404 - File not found.");
	    }
	    
	}

Its purpose is open a serial port using the serialport node package and emit each serial read to the client using serial.io. 

Update the serial path **"/dev/ttyUSB0"** to whichever port your device is attached to. For example, the code in this example was run on Linux, but on my Windows machine the path was **"COM4"**.

It also sets up a web server to host our visualization. If you use localhost and port 5000 with the code below, use the url **http://localhost:5000/threejs** to access the visualization.

Set up three.html as below:

	<html>
		<head>
			<title>POUR ME WHISKEY</title>
		    <style type="text/css">
	            body {
	                    font-family: Monospace;
	                    background-color: #f0f0f0;
	                    margin: 0px;
	                    overflow: hidden;
	                }
	        </style>
		</head>
		<body>
			<script src="lib/three.min.js"></script>
	        <script src="lib/Projector.js"></script>
	        <script src="lib/CanvasRenderer.js"></script>
	        <script src="lib/jquery-2.1.1.min.js"></script>
	        <script src="lib/socket.io.js"></script>
	        <script src="generateThree.js"></script>
		</body>
	</html>

The first three script references are all part of three.js. Check their repository on GitHub for the source files. **generateThree.js** will hold all of our custom javascript to set up the render:

	/*
	
	Generate 3D render using serial data from IMU
	
	*/
	
	'use strict';
	
	// Declare required variables
	var dataRollx = 0;
	var dataRolly = 0;
	var dataRollz = 0;
	var dataRollxArray = [];
	var dataRollyArray = [];
	var dataRollzArray = [];
	var accuracy = 2;
	var orderOfMag = (Math.PI/180);
	var container;
	var camera, scene, renderer;
	var cube, plane;
	var targetRotation = 0;
	var targetRotationOnMouseDown = 0;
	var windowHalfX = window.innerWidth / 2;
	var windowHalfY = window.innerHeight / 2;
	
	//Connect to socket.io
	var serverIP = "localhost";
	var socket = io.connect(serverIP + ':5000');
	console.log('socket connected to: ' + serverIP);
	
	// Start reading IMU data
	runSocket();
	init();
	animate();
	
	function runSocket() {
	        socket.on('serial_update', function(data) {
	            if (data.charAt(0) === 'O') {
	                console.log(data);
	                var dataArray = data.split(/ /);
	
	                // set x
	                dataRollx = (dataArray[1] *= orderOfMag).toFixed(accuracy);
	                
	                // set y
	                dataRolly = (dataArray[2] *= orderOfMag).toFixed(accuracy);
	
	                // set z
	                dataRollz = (dataArray[3] *= orderOfMag).toFixed(accuracy);
	
	                console.log(dataRollx + "," + dataRolly + "," + dataRollz);
	            }
	        });
	}
	
	function init() {
	
	    container = document.createElement( 'div' );
	    document.body.appendChild( container );
	
	    var info = document.createElement( 'div' );
	    info.style.position = 'absolute';
	    info.style.top = '10px';
	    info.style.width = '100%';
	    info.style.textAlign = 'center';
	    info.innerHTML = 'Visualize IMU';
	    info.setAttribute('id', 'pourHeading');
	    container.appendChild( info );
	
	    $("#pourHeading").append("<div id='subHeading'></div>");
	
	    // Set up camera
	    camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 1, 1000 );
	    camera.position.y = 150;
	    camera.position.z = 500;
	
	    scene = new THREE.Scene();
	
	    // Create cube
	    var geometry = new THREE.BoxGeometry( 200, 200, 200 );
	
	    for ( var i = 0; i < geometry.faces.length; i += 2 ) {
	
	        var hex = Math.random() * 0xffffff;
	        geometry.faces[ i ].color.setHex( hex );
	        geometry.faces[ i + 1 ].color.setHex( hex );
	
	    }
	
	    var material = new THREE.MeshBasicMaterial( { vertexColors: THREE.FaceColors, overdraw: 0.5 } );
	
	    cube = new THREE.Mesh( geometry, material );
	    cube.position.y = 150;
	    scene.add( cube );
	
	    // Create background plane
	    var geometry = new THREE.PlaneBufferGeometry( 400, 200 );
	    geometry.applyMatrix( new THREE.Matrix4().makeRotationX( - Math.PI / 2 ) );
	
	    var material = new THREE.MeshBasicMaterial( { color: 0xe0e0e0, overdraw: 0.5 } );
	
	    plane = new THREE.Mesh( geometry, material );
	    scene.add( plane );
	
	    renderer = new THREE.CanvasRenderer();
	    renderer.setClearColor( 0xf0f0f0 );
	    renderer.setSize( window.innerWidth, window.innerHeight );
	    container.appendChild( renderer.domElement );
	
	    window.addEventListener( 'resize', onWindowResize, false );
	}
	
	function onWindowResize() {
	        windowHalfX = window.innerWidth / 2;
	        windowHalfY = window.innerHeight / 2;
	
	        camera.aspect = window.innerWidth / window.innerHeight;
	        camera.updateProjectionMatrix();
	
	        renderer.setSize( window.innerWidth, window.innerHeight );
	}
	
	function animate() {
	        requestAnimationFrame( animate );
	        render();
	}
	
	function render() {
	    cube.rotation.x = -dataRollx;
	    cube.rotation.y = -dataRollz;
	    cube.rotation.z = -dataRolly;
	    renderer.render( scene, camera );
	}

Each reading is passed to the client through *socket.on('serial_update')*. The string that is passed is then parsed into an array (*dataArray*) and values stored as *dataRollx*, *dataRolly*, *dataRollz*. The scene and cube are generated. Finally in *render()*, *cube.rotation* is set for each axis with dataRoll values. You may need to adjust the sign of the dataRoll values (negative or positive) depending on the orentation of your IMU. Try switching signs if you have inverted movements.