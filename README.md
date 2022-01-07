<h1>BigCommerce Stencil Cornerstone Theme</br></h1>
<h2>URL: <a href="https://bigstore4.mybigcommerce.com/">BigPage</a></h2>
<h2>Preview code: p68n3l7sy8</h2>
<p>
  <h2>card</h2>
  <ul>
  <li>https://github.com/fxspec06/BigPage/blob/master/templates/components/products/card.html</li>
    <li>added image hover to the data-src attribute using data-hoverimage attribute</li>
    <li>added mouseover and mouseout functions (jquery hover was doing nothing)</li>
    <li>had to use event mouseover and mouseout on the card class, and not card-figure, so hover doesn't change unless the mouse pointer is directly on the image</li>
  </ul>
  </p>
</br>
</br>
</br>
</br>
<p>
<h2>bigcommerce.http</h2>
<ul><li>https://github.com/fxspec06/BigPage/blob/master/bigcommerce.http</li>
  <li>this file used to create requests used to create and update the entire store</li>
  <li>using the REST API in Visual Studio Code, you can click on the link above each command to run it, and VSCode displays the results neatly</li>
  </ul>
</p>
</br>
</br>
</br>
</br>
<p>
<h2>category.html</h2>
<ul>
  <li>added two buttons to top of all category pages</li>
  <li>the first is add all to cart, which always displays</li>
<li>the second is remove all from cart, which was a bit trickier to get working, but is dynamic in that it only displays if there are items in the cart</li>
<li>for both of these buttons, the more items to add or remove, the longer, because jquery ajax only allows one instance at a time, and if you're adding 12 requests that's 12 times like .7 seconds</li>
  <li>this is the best way (maybe the only way) to accomplish this i would think</li>
  <li>https://github.com/fxspec06/BigPage/blob/master/templates/pages/category.html</li>
</ul>
  </p>
<p>
  <h2>base.html</h2>
  <ul>
  <li>https://github.com/fxspec06/BigPage/blob/master/templates/layout/base.html</li>
  <li>added working jquery using external url</li>
  </ul>
</p>
