---
title: Using Optimizely with React
keywords: Optimizely, React, AbTesting, JavaScript
date: 2016-07-31 16:34:00
tags:
- Optimizely
- React
- JavaScript
---

Optimizely is an A/B testing tool used to test different variations of the same page/component to see which converts better.

Let's say you have an ecommerce website with a product grid. The products in the grid currently contain only minimal information about it - name, picture, price and Buy button. Let's say you have an hypothesis that adding ratings and other details such as weight/size etc would make more people click on the Buy button. But you're concerned that adding too much information would clutter the UI and drive away customers. One way of solving this would be to do an A/B test between the existing UI (called as "Control" in A/B testing world) and the new UI with additional details (called "Variation") and see which has more people clicking on the Buy button ("Conversion"). The traffic to the website is split into two - one group would always see the the Control and other would always see the Variation.

In Optimizely, the above is called an "Experiment" and both "Control" and "Variation" are the experiment's "Variations". Once you create an experiment and its variations, you'll be shown an visual editor where you can customise the appearance of each variation by modifying/rearranging the elements in the page. The visual editor then translates those cutomisations into jQuery code called `Variation Code`. Depending on which variation an user is grouped into, the Optimizely library loads the appropriate `Variation Code` and thus displaying different UIs.

This workflow works well for static websites, but if the website is dynamic and uses React, then letting the `Variation Code` do arbitrary DOM manipulations doesn't look like a good idea.

One solution is to create different components for each variation and using a container component to render the correct variation. But before that we need to know which variation the user belongs to and if the experiment is running (active) or not. Fortunately, Optimizely exposes [Data Object](http://developers.optimizely.com/javascript/reference/#the-data-object) which can be used to get the above data. We can use `window.optimizely.data.state.activeExperiments` to get the list of all running experiments and `window.optimizely.data.state.variationIdsMap[<experimentId>][0]` to get the variation the user belongs to.

```javascript
// abtest-container.js

var AbTestContainer = React.createClass({

	propTypes: {
		experimentId: React.PropTypes.string.isRequired,
		defaultVariationId: React.PropTypes.string.isRequired,
		variations: React.PropTypes.arrayOf(React.PropTypes.shape({
			variationId: React.PropTypes.string.isRequired,
			component: React.PropTypes.element.isRequired
		}))
	},

	getVariation: function () {
		// Make sure you perform appropriate guard checks before using this in production!
		var activeExperiments = window.optimizely.data.state.activeExperiments;
		var isExperimentActive = _.contains(activeExperiments, this.props.experimentId);
		var variation, variationId;

		if (isExperimentActive) {
			variationId = window.optimizely.data.state.variationIdsMap[experimentId][0];
		} else {
			variationId = this.props.defaultVariationId;
		}

		variation = _.findWhere(this.props.variations, { variationId: variationId });
		return variation;
	},

	render: function () {
		return this.getVariation();
	}

});

```

And this can be used as follows

```javascript
// products-page.js

...

render: function () {
	var variations = [
		{ variationId: '111', component: <ProductGrid/> },
		{ variationId: '222', component: <ProductGridWithAdditionalDetails/> }
	];

	return (
		<AbTestContainer
			experimentId='000'
			defaultVariationId='111'
			variations={variations}
		/>
	);
}

...

```

The IDs for experiment and variations would be present in the visual editor page under "Options -> Diagnostic Report".
