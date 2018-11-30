This is a walkthrough of how to get started with Selenium Grid. In doing so we will be setting up Docker on Digital Ocean to keep it simple and relevent. Then we will set up the grid and show you how to use it and scale it up for further simultaneous testing.

**Why use Selenium Grid**
To take Selenium testing forward, especially with a large suite of tests built up, the point comes where it is wise to adopt the practice of parallelization - running multiple test cases simultaneously to achieve this we turn to Selenium-Grid. 

With this we are able to easily scale up our testing as the tests themselves do not need to be rewriten, the key change will be to just change the designated webdriver to the new one. 

**Docker**
A good way of setting up Selenium-Grid is by using Docker. With Docker it is easy to setup and deploy containers which makes brings with it many advantages, primarily that it is easy to scale up and to replicate across teams with ease. Following the instructions below will work every time in the same way, avoiding any hasstle with setting up multiple VPS's as our nodes. 


**Get starting with Digital Ocean**
An easy way to get start with a hosted solution is to use a service like Digital Ocean.

A benefit of using Digital Ocean is that it is possible to make use of the 'One-click apps', from here it possible to spin up a Docker instance in a Ubuntu VPS(Droplet). 
![IMG1](https://github.com/john-lock/SeleniumGrid/blob/master/images/img1.png)

Once successful we should see something like this when we look at the details of the instance:
![IMG2](https://github.com/john-lock/SeleniumGrid/blob/master/images/img2.png)

Note; I recommend starting out on the smallest droplet size to get things going initially, later it is possible to scale up the resources depending on work done. Scaling down resources is however not quite so straightforward as users cannot downscale to droplets with smaller storage capacities. 


# Create the Selenium Hub 
Now it is time get get Selenium running, this starts with getting a Hub started which we will connect to with each of our nodes - which represents a different browsers to test in. 

To do so, we create a docker-compose.yml file containing the following:
```yml
selhub:
  image: selenium/hub
  ports:
    - 4444:4444

nodeffox:
  image: selenium/node-firefox-debug
  ports:
    - 5900
  links:
    - selhub:hub

nodechrome:
  image: selenium/node-chrome-debug
  ports:
    - 5900
  links:
    - selhub:hub

```

This will setup our Hub along with both a Firefox and Chrome nodes. To start the container run `docker-compose up -d`
The images will then be downloaded each of the 3 containers (1 Hub and 2 Nodes). Once this is complete you can verify this with `docker ps`

Now it is possible to see the Grid in the browser by going to `<VPS ip>:4444/grid/console` where you should see something like this:
![IMG3](https://github.com/john-lock/SeleniumGrid/blob/master/images/img3.png)

# Scale up a node
To run multiple tests in parallel on a node, the node will need to be scaled up. This can be used to further reduce the time taken to run a suite of tests. To do this is simple using the following command:
`docker-compose up -d --scale nodeffox=2`
Once this is run you will see the upscaled nodes in /grid/console:
![IMG4](https://github.com/john-lock/SeleniumGrid/blob/master/images/img4.png)

# Connect to/Use in a practical example
We are now all set to put this new Grid to use. When using Selenium(Python) we must declare our webdriver, in a simple example with Pythons built in Unittest module this might be in the form of:
```python
...
class ImportantFeatureTest(unittest.TestCase):

    def setUp(self):
        self.driver = webdriver.Chrome()
...

```
However to use our new Selenium Grid we will amend this to the following:
```python
...
class ImportantFeatureTest(unittest.TestCase):

    def setUp(self):
        self.driver = webdriver.Remote(
            command_executor='http//<VPS IP>:4444/wd/hub',
            desired_capabilities={'browserName': 'firefox', 'javascriptEnabled': True})
...
```
You can now use the Grid as you would a standard Selenium driver but enjoy the cross browser options and better performance!

# Conclusion 
Hopefully this has been useful to demonstrate how to get going with Selenium Grid. From experience I know that the switch to parallel testing can be a big one which discourages many. However using Docker in this way make this very straightforward to get going in a way you will feel comfortable with even if without experience using Docker before.

Next steps which I hope to add here:
- How to watch your tests being carried out, and take screenshots on failure, with VNC. 
- To scale up even further, we can utilize Docker Swarm by deploying our Grid across multiple machines as a cluster.