# Serving Raspberry Pi with Flask

The following is an adapted excerpt from [Getting Started with Raspberry Pi](http://www.amazon.com/Getting-Started-Raspberry-Matt-Richardson/dp/1449344216/?tag=mattricha-20).
I especially like this section of the book because it shows off one of the strengths of the Pi: its ability to combine modern web frameworks with hardware and electronics.

Not only can you use the Raspberry Pi to get data from servers via the internet, but your Pi can also act as a server itself.
There are many different web servers that you can install on the Raspberry Pi.
Traditional web servers, like Apache or lighttpd, serve the files from your board to clients.
Most of the time, servers like these are sending HTML files and images to make web pages, but they can also serve sound, video, executable programs, and much more.

However, there's a new breed of tools that extend programming languages like Python, Ruby, and JavaScript to create web servers that dynamically generate the HTML when they receive HTTP requests from a web browser.
This is a great way to trigger physical events, store data, or check the value of a sensor remotely via a web browser.
You can even create your own JSON API for an electronics project!

## Flask Basics

We're going to use a Python web framework called [Flask](http://flask.pocoo.org/) to turn the Raspberry Pi into a dynamic web server.
While there's a lot you can do with Flask "out of the box," it also supports many different extensions for doing things such as user authentication, generating forms, and using databases.
You also have access to the wide variety of standard Python libraries that are available to you.

In order to install Flask, you'll need to have pip installed.
If you haven't already installed pip, it's easy to do:

	pi@raspberrypi ~ $ sudo apt-get install python-pip

After pip is installed, you can use it to install Flask and its dependencies:


	pi@raspberrypi ~ $ sudo pip install flask

To test the installation, create a new file called `hello-flask.py` with the code from below.

### `hello-flask.py`

	from flask import Flask
	app = Flask(__name__)

	@app.route("/")
	def hello():
		return "Hello World!"

	if __name__ == "__main__":
		app.run(host='0.0.0.0', port=80, debug=True)


Here's a breakdown of each line of code:

	from flask import Flask

Load the Flask module into your Python script

	app = Flask(__name__)

Create a Flask object called app

	@app.route("/")
		def hello():

Run the code below this function when someone accesses the root URL of the server

	return "Hello World!"

Send the text "Hello World!" to the client's web browser

	if __name__ == "__main__":

If this script was run directly from the command line

	app.run(host='0.0.0.0', port=80, debug=True)

Have the server listen on port 80 and report any errors.


**NOTE:** Before you run the script, you need to know your Raspberry Pi's IP address. You can run `ifconfig` to find it.
An alternative is to install avahi-daemon (run `sudo apt-get install avahi-daemon` from the command line).
This lets you access the Pi on your local network through the address http://raspberrypi.local.
If you're accessing the Raspberry Pi web server from a Windows machine, you may need to put [Bonjour Services](http://support.apple.com/kb/DL999) on it for this to work.

Now you're ready to run the server, which you'll have to do as root:

	pi@raspberrypi ~ $ sudo python hello-flask.py
	 * Running on http://0.0.0.0:80/
	 * Restarting with reloader

From another computer on the same network as the Raspberry Pi, type your Raspberry Pi's IP address into a web browser.
If your browser displays "Hello World!", you know you've got it configured correctly.
You may also notice that a few lines appear in the terminal of the Raspberry Pi:

	10.0.1.100 - - [19/Nov/2012 00:31:31] "GET / HTTP/1.1" 200 -
	10.0.1.100 - - [19/Nov/2012 00:31:31] "GET /favicon.ico HTTP/1.1" 404 -

The first line shows that the web browser requested the root URL and our server returned HTTP status code 200 for "OK."
The second line is a request that many web browsers send automatically to get a small icon called a _favicon_ to display next to the URL in the browser's address bar.
Our server doesn't have a `favicon.ico` file, so it returned HTTP status code 404 to indicate that the URL was not found.

## Templates

If you want to send the browser a site formatted in proper HTML, it doesn't make a lot of sense to put all the HTML into your Python script.
Flask uses a template engine called [Jinja2](http://jinja.pocoo.org/docs/templates/) so that you can use separate HTML files with placeholders for spots where you want dynamic data to be inserted.

If you've still got `hello-flask.py` running, press Control-C to kill it.

To make a template, create a new file called `hello-template.py` with the code from below. In the same directory with `hello-template.py`, create a subdirectory called `templates`.
In the `templates` subdirectory, create a file called `main.html` and insert the code from below.
Anything in double curly braces within the HTML template is interpreted as a variable that would be passed to it from the Python script via the `render_template` function.

### `hello-template.py`

	from flask import Flask, render_template
	import datetime
	app = Flask(__name__)

	@app.route("/")
	def hello():
	   now = datetime.datetime.now()
	   timeString = now.strftime("%Y-%m-%d %H:%M")
	   templateData = {
	      'title' : 'HELLO!',
	      'time': timeString
	      }
	   return render_template('main.html', **templateData)

	if __name__ == "__main__":
	   app.run(host='0.0.0.0', port=80, debug=True)

Let's take a look at some of the important lines of code.

	now = datetime.datetime.now()

Get the current time and store it in the object `now`

	timeString = now.strftime("%Y-%m-%d %H:%M")

Create a formatted string using the date and time from the `now` object

	templateData = {
	   'title' : 'HELLO!',
	   'time': timeString
	   }

Create a _dictionary_ of variables (a set of _keys_, such as `title` that are associated with values, such as `HELLO!`) to pass into the template

	return render_template('main.html', **templateData)

Return the `main.html` template to the web browser using the variables in the `templateData` dictionary

### `main.html`

	<!DOCTYPE html>
	   <head>
	      <title>{{ "{{ title " }}}}</title>
	   </head>

	   <body>
	      <h1>Hello, World!</h1>
	      <h2>The date and time on the server is: {{ "{{ time " }}}}</h2>
	   </body>
	</html>

In `main.html`, any time you see a word within double curly braces, it'll be replaced with the matching key's value from the `templateData` dictionary in `hello-template.py`.

Now, when you run `hello-template.py` (as before, you need to use `sudo` to run it) and pull up your Raspberry Pi's address in your web browser, you should see a formatted HTML page with the title "HELLO!" and the Raspberry Pi's current date and time.

**NOTE:** While it's dependent on how your network is set up, it's unlikely that this page is accessible from outside your local network via the Internet.
If you'd like to make the page available from outside your local network, you'll need to configure your router for port forwarding.
Refer to your router's documentation for more information about how to do this.

## Connecting the Web to the Real World
You can use Flask with other Python libraries to bring additional functionality to your site.
For example, with the [RPi.GPIO](https://pypi.python.org/pypi/RPi.GPIO) Python module you can create a website that interfaces with the physical world.
To try it out, hook up a three buttons or switches to pins 23, 24, and 25 as shown in this graphic:

![Connecting three buttons to Raspberry Pi]({{site.url}}/images/Raspi-Buttons.png)

The following code expands the functionality of `hello-template.py`, so copy it to a new file called `hello-gpio.py`.
Add the RPi.GPIO module and a new _route_ for reading the buttons, as I've done in the code below.
The new route will take a variable from the requested URL and use that to determine which pin to read.

### `hello-gpio.py`

	from flask import Flask, render_template
	import datetime
	import RPi.GPIO as GPIO
	app = Flask(__name__)

	GPIO.setmode(GPIO.BCM)

	@app.route("/")
	def hello():
	   now = datetime.datetime.now()
	   timeString = now.strftime("%Y-%m-%d %H:%M")
	   templateData = {
	      'title' : 'HELLO!',
	      'time': timeString
	      }
	   return render_template('main.html', **templateData)

	@app.route("/readPin/<pin>")
	def readPin(pin):
	   try:
	      GPIO.setup(int(pin), GPIO.IN)
	      if GPIO.input(int(pin)) == True:
	         response = "Pin number " + pin + " is high!"
	      else:
	         response = "Pin number " + pin + " is low!"
	   except:
	      response = "There was an error reading pin " + pin + "."

	   templateData = {
	      'title' : 'Status of Pin' + pin,
	      'response' : response
	      }

	   return render_template('pin.html', **templateData)


	if __name__ == "__main__":
	   app.run(host='0.0.0.0', port=80, debug=True)

Here are the highlights:

	@app.route("/readPin/<pin>")

Add a dynamic route with pin number as a variable.

	   try:
	      GPIO.setup(int(pin), GPIO.IN)
	      if GPIO.input(int(pin)) == True:
	         response = "Pin number " + pin + " is high!"
	      else:
	         response = "Pin number " + pin + " is low!"
	   except:
	      response = "There was an error reading pin " + pin + "."

If the code indented below `try` raises an exception (that is, there's an error), run the code in the `except` block.

	GPIO.setup(int(pin), GPIO.IN)

Take the pin number from the URL, convert it into an integer and set it as an input

	if GPIO.input(int(pin)) == True:
		response = "Pin number " + pin + " is high!"

If the pin is high, set the response text to say that it's high

	else:
		response = "Pin number " + pin + " is low!"

Otherwise, set the response text to say that it's low

	except:
		response = "There was an error reading pin " + pin + "."

If there was an error reading the pin, set the response to indicate that

You'll also need to create a new template called `pin.html`.
It's not very different from `main.html`, so you may want to copy `main.html` to `pin.html` and make the appropriate changes as shown

### `pin.html`

	<!DOCTYPE html>
	   <head>
	      <title>{{ "{{ title " }}}}</title>
	   </head>

	   <body>
	      <h1>Pin Status</h1>
	      <h2>{{ "{{ response " }}}}</h2>
	   </body>
	</html>


With `hello-gpio.py` running, when you point your web browser to your Raspberry Pi's IP address, you should see the standard "Hello World!" page we created before.
But add `/readPin/24` to the end of the URL, so that it looks something like `http://10.0.1.103/readPin/24`.
A page should display showing that the pin is being read as low.
Now hold down the button connected to pin 24 and refresh the page; it should now show up as high!

Try the other buttons as well by changing the URL.
The great part about this code is that we only had to write the function to read the pin once and create the HTML page once, but it's almost as though there are separate webpages for each of the pins!

## Project: WebLamp
In chapter 7 of [Getting Started with Raspberry Pi](http://www.amazon.com/Getting-Started-Raspberry-Matt-Richardson/dp/1449344216/?tag=mattricha-20), Shawn and I show you how to use Raspberry Pi as a simple AC outlet timer using the Linux job scheduler, `cron`.
Now that you know how to use Python and Flask, you can now control the state of a lamp over the web.
This basic project is simply a starting point for creating Internet-connected devices with the Raspberry Pi.

And just like how the previous Flask example showed how you can have the same code work on multiple pins, you'll set up this project so that if you want to control more devices in the future, it's easy to add.

Connect a Power Switch Tail  II to pin 25 of the Raspberry Pi, as shown in the illustration below. If you don't have a Power Switch Tail, you can connect an LED to the pin to try this out for now.

![Conencting Power Switch Tail to Raspberry Pi]({{site.url}}/images/raspi-power-tail.jpg)

If you have another PowerSwitch Tail II relay, connect it to pin 24 to control a second AC device.
Otherwise, just connect an LED to pin 24.
We're simply using it to demonstrate how multiple devices can be controlled with the same code.


Create a new directory in your home directory called `WebLamp`.

In `WebLamp`, create a file called `weblamp.py` and put in the code from below:

### `weblamp.py`

	import RPi.GPIO as GPIO
	from flask import Flask, render_template, request
	app = Flask(__name__)

	GPIO.setmode(GPIO.BCM)

	# Create a dictionary called pins to store the pin number, name, and pin state:
	pins = {
	   24 : {'name' : 'coffee maker', 'state' : GPIO.LOW},
	   25 : {'name' : 'lamp', 'state' : GPIO.LOW}
	   }

	# Set each pin as an output and make it low:
	for pin in pins:
	   GPIO.setup(pin, GPIO.OUT)
	   GPIO.output(pin, GPIO.LOW)

	@app.route("/")
	def main():
	   # For each pin, read the pin state and store it in the pins dictionary:
	   for pin in pins:
	      pins[pin]['state'] = GPIO.input(pin)
	   # Put the pin dictionary into the template data dictionary:
	   templateData = {
	      'pins' : pins
	      }
	   # Pass the template data into the template main.html and return it to the user
	   return render_template('main.html', **templateData)

	# The function below is executed when someone requests a URL with the pin number and action in it:
	@app.route("/<changePin>/<action>")
	def action(changePin, action):
	   # Convert the pin from the URL into an integer:
	   changePin = int(changePin)
	   # Get the device name for the pin being changed:
	   deviceName = pins[changePin]['name']
	   # If the action part of the URL is "on," execute the code indented below:
	   if action == "on":
	      # Set the pin high:
	      GPIO.output(changePin, GPIO.HIGH)
	      # Save the status message to be passed into the template:
	      message = "Turned " + deviceName + " on."
	   if action == "off":
	      GPIO.output(changePin, GPIO.LOW)
	      message = "Turned " + deviceName + " off."
	   if action == "toggle":
	      # Read the pin and set it to whatever it isn't (that is, toggle it):
	      GPIO.output(changePin, not GPIO.input(changePin))
	      message = "Toggled " + deviceName + "."

	   # For each pin, read the pin state and store it in the pins dictionary:
	   for pin in pins:
	      pins[pin]['state'] = GPIO.input(pin)

	   # Along with the pin dictionary, put the message into the template data dictionary:
	   templateData = {
	      'message' : message,
	      'pins' : pins
	   }

	   return render_template('main.html', **templateData)

	if __name__ == "__main__":
	   app.run(host='0.0.0.0', port=80, debug=True)

Since there's a lot going on in this code, I've placed on the explanations inline, as comments.

Create a new directory within `WebLamp` called `templates`.

Inside `templates`, create the file `main.html`:

### `main.html`
{% raw %}

	<!DOCTYPE html>
	<head>
	   <title>Current Status</title>
	</head>

	<body>
	   <h1>Device Listing and Status</h1>

	   {% for pin in pins %}
	   <p>The {{ pins[pin].name }}
	   {% if pins[pin].state == true %}
	      is currently on (<a href="/{{pin}}/off">turn off</a>)
	   {% else %}
	      is currently off (<a href="/{{pin}}/on">turn on</a>)
	   {% endif %}
	   </p>
	   {% endfor %}

	   {% if message %}
	   <h2>{{ message }}</h2>
	   {% endif %}

	</body>
	</html>

{% endraw %}

There are two templating concepts being illustrated in this file. First:

{% raw %}

	{% for pin in pins %}
	<p>The {{ pins[pin].name }}
	{% if pins[pin].state == true %}
	   is currently on (<a href="/{{pin}}/off">turn off</a>)
	{% else %}
	   is currently off (<a href="/{{pin}}/on">turn on</a>)
	{% endif %}
	</p>
	{% endfor %}

{% endraw %}

This sets up a for loop to cycle through each pin, print its name and its state. It then gives the option to change its state, based on its current state. Second:

{% raw %}

	   {% if message %}
	   <h2>{{ message }}</h2>
	   {% endif %}

{% endraw %}

This if statement will print a message passed from your Python script if there's a message set (that is, a change happened).

In terminal, navigate to the `WebLamp` directory and start the server (be sure to use Control-C to kill any other Flask server you have running first):

	pi@raspberrypi ~/WebLamp $ sudo python weblamp.py

![The device interface, as viewed through a phone's web browser]({{site.url}}/images/Flask-phone.png)

The best part about writing the code in this way is that you can very easily add as many devices that the hardware will support.
Simply add the information about the device to the pins dictionary.
When you restart the server, the new device will appear in the status list and its control URLs will work automatically.

There's another great feature built in: if you want to be able to flip the switch on a device with a single tap on your phone, you can create a bookmark to the address `http://ipaddress/pin/toggle`.
That URL will check the pin's current state and switch it.

For more information about using Raspberry Pi, check out [Getting Started with Raspberry Pi](http://www.amazon.com/Getting-Started-Raspberry-Matt-Richardson/dp/1449344216/?tag=mattricha-20).
