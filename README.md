
**Why use Selenium Grid**
To take Selenium testing forward at some point, especially with a large suite of tests built up, the point comes where it is wise to adopt the practice of parallelization. To achieve this we turn to Selenium-Grid …

- Why Docker
A good way of setting up Selenium-Grid is by using Docker. With Docker it is easy to setup and deploy containers which makes brings with it many advantages, primarily that it is easy to scale up and …


- Get starting with Digital Ocean
An easy way to get start with a hosted solution is to use a service like Digital Ocean.

A benefit of using Digital Ocean is that it is possible to make use of the 'One-click apps', from here it possible to spin up a Docker instance in a Ubuntu VPS(Droplet). 
![img1] (https://github.com/john-lock/selenium-grid/images/img1.png)

Once successful we should see something like this when we look at the details of the instance:
img2


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
img3

# Scale up a node
To run multiple tests in parallel on a node, the node will need to be scaled up. This can be used to further reduce the time taken to run a suite of tests. To do this is simple using the following command:
`docker-compose up -d --scale nodeffox=2`
Once this is run you will see the upscaled nodes in /grid/console:
img4

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
Next steps:
- VNC
- Docker Swarm