# Tutorial for Google Enhanced e-Commerce Tracking in plentymarkets
### Version 1.0
To implement (almost) full functional enhanced e-commerce tracking, go through the following steps.

### Warning
Never do any changes in your running production enviroment. It is highly recommended to setup a new layout for this guide and please backup
your current active layout.

### Requirements
  - first, be sure to have UniversalAnalytics (analytics.js) running
  - you need a running plentymarkets online shop, of course
  - the "new" fully inidvidual checkout is required

## Google Analytics setup
1. You need to activate the **Enhanced E-Commerce Setting** as you can see in the image below
![setup enhanced ecommerce](https://raw.githubusercontent.com/cytRasch/plenty-enhancedecommerce/master/ga-01.png)
 
2. After activation you nee to setup your checkout-matching funnels like this
![setup enhanced ecommerce](https://raw.githubusercontent.com/cytRasch/plenty-enhancedecommerce/master/ga-02.png)

**Be careful if you PayPal Plus, for example. In this case you have to split Payment & Shipping**\
plentymarkets manual for this: [https://www.plentymarkets.eu/handbuch/payment/paypal-plus/](https://www.plentymarkets.eu/handbuch/payment/paypal-plus/)

## plentymarkets setup
### 1. $PageDesignContent
1. Okay, now copy & paste your js-Tracking snippet from Google Analytics somewhere inside the `head`-Tag.
2. Add some needed plugins from Google Analytics that your code looks similar to this:
```
<!-- ga -->
<script type="text/javascript">
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
	(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
	m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
	})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
	
    // ga init
	ga('create', 'UA-12345678', 'your-shop.de');
    
    // ga plugins
	ga('require', 'linkid');
	ga('require', 'ec');
	ga('set', 'forceSSL', true);
	ga('set', 'anonymizeIp', true);
	ga('set', '&cu', 'EUR');
</script>
```

3. Now we have to integrate the `pageview`-snippet to send all needed data to google. please add the following snippet (if possible) right before the `</body>`-Tag
````
<script type="text/javascript">
    ga('send','pageview');
</script>
````

#### 1.1 $
