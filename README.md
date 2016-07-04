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

#### 1.1 $ItemViewCategoriesList
This is getting a little bit more complex. Here you can handle all activities like:
* addImpression
* addProduct
* and setAction

1. Right after the `$FormOpenOrder`-Tag in your for-loop you have to copy & paste this:
```
<!-- ga -->
<script type="text/javascript">
    {% 
        # dynamic function naming
        $_fnName = 'onProductClick'.$ID;
    %}
    
    // send impression/view of products
    ga('ec:addImpression', {
        'id': '$ID',
        // send the clean name without any html-tags
        'name': '{% trim(strip_tags($Name[1])) %}',
        'category': '$CurrentCategoryName',
        'list': 'CategoryView',
        'position': $RowCount
    });
    
    // create dynamic function for adding products
    function $_fnName() {
        ga('ec:addProduct', {
            'id': '$ID',
            'name': '{% trim(strip_tags($Name[1])) %}',
            'category': '$CurrentCategoryName',
            'brand': '$Producer',
            'position': $RowCount,
            'price': '{% number_format($Price, 2, '.', '') %}'
        });
        ga('ec:setAction', 'click', {list: 'CategoryView'});
        
        // if you click on a product, say it to google
        ga('send', 'event', 'CategoryView', 'click', 'Results', {
            hitCallback: function() {
                document.location = '{% Link_Item($ID) %}';
            }
        });
    }
    
    // the document ready stuff (jQuery required)
    $(document).ready(function() {
        $('#identifier-$ID .PlentyWebshopButton').on('click',function(event) {
            ga('ec:addProduct', {
                'id': '$ID',
                'name': '{% trim(strip_tags($Name[1])) %}',
                'category': '$CurrentCategoryName',
                'brand': '$Producer',
                'price': '{% number_format($Price, 2, '.', '') %}',
                // this is for qty = 1, if you have a qty input on your
                // category list you have to add more functionality
                'quantity': 1
            });
            ga('ec:setAction', 'add');
            ga('send', 'event', 'CategoryView', 'click', 'Add to Basket');
        });
    });
</script>
```

2. Now we have to add some functions to the product if the user interacts
* add a unique identifier to the product-wrapping html-tag, for example:
`id="identifier-$ID"`
```
<!-- item box -->
<li class="col-xs-12 col-sm-3 col-md-4 col-lg-3 margin-bottom-2 center itemBox tileView onHover action-$ActionId" data-plenty-id="$ID" id="identifier-$ID">
	$FormOpenOrder
	...
	$FormCloseOrder
</li>
```
* Find all `<a>`-Tags with a link to the product page and add a `onclick`-function referenced to our generated function
```
<a class="name block" onclick="onProductClick$ID(); return !ga.loaded;" href="{% Link_Item($ID) %}">...</a>
```
* At last, find the button for adding your product to the basket and wrap it with an unique ID that it looks similar to this:
```
<div class="buttonBox isAddToBasket" id="addToBasket-$ID">
	<button class="btn btn-primary text-center" data-plenty="click:Basket.addBasketItem(this)">
		<span class="glyphicon glyphicon-shopping-cart"></span>in den Warenkorb
	</button>
</div>
```

#### 1.2 $ItemViewSearchResultsList
This is almost the same procedure as you did with $ItemViewCategoriesList. Just change your reference from **CategoryView** to **SearchResult**

#### 1.3 $ItemViewSingleItem
1. At the very beginning simply add this short snippet:
```
<!-- ga -->
<script type="text/javascript">
    ga('ec:addProduct', {
        'id': '$ID',
        'name': '{% trim(strip_tags($Name[1])) %}',
        'category': '$CurrentCategoryName',
        'brand': '$Producer',
        'price': '{% number_format($Price, 2, '.', '') %}'
    });
    ga('ec:setAction', 'ItemView');
    
    $(document).ready(function(){
       
        $('.toBasketWrapper button').on('click',function(event) {
            ga("ec:addProduct", {
                "id": "$ID",
                'name': '{% trim(strip_tags($Name[1])) %}',
                "price": "{% number_format($Price, 2, '.', '') %}",
                "brand": "$Producer",
                "category": "$CurrentCategoryName",
                "position": 0,
                "quantity": $('.quantityInput').val()
            });
            ga("ec:setAction", "add");
            ga("send", "event", "ItemView", "click", "Add to Basket");
        });
        
    });
</script>
```

Well done, that's it. Now we can got setup the checkout-proccess.
