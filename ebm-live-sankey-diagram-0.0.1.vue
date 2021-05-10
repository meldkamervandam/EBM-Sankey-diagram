<template>
  <div class="card">
    <div class="card-header" v-if="showHeader">
      <h3 class="card-title" v-if="title">{{ title }}</h3>
      <h4 class="card-subtitle" v-if="header.subtitle">
        {{ subtitle }}
      </h4>
    </div>
	<div
      v-bind:class="{ shallow: isShallow }"
      class="card-content"
      ref="chart"
    ></div>
  </div>
</template>

<script>
import * as am4core from '@amcharts/amcharts4/core';
import * as am4charts from '@amcharts/amcharts4/charts';
import {
  metricHasQueryRequirements,
  mapMetricInputToQuery,
} from '@/shared/utils';

export default {
  name: 'live-sankey-diagram',
  props: { contextFactory: Function },
  computed: {
	header() {
	  return this.context.inputs.title || this.context.inputs;
	},
	hideHeader() {
	  return this.height !== null ? this.height <= 130 : false;
	},
	showHeader() {
	  return (this.context.inputs.title || this.context.inputs.subtitle) && !this.hideHeader;
	},
	isShallow() {
	  return this.height !== null ? this.height <= 180 : false;
	},
  },
  created() {
    this.context = this.contextFactory();
    this.client = this.context.createLoggingDataClient();
    
	//Get all created nodes
	nodes = this.context.nodes;
	let sankeydatas = [];
	
	//Get color and push in array
	for(j=0; j<nodes.length; j++){
		//Get last source metric
		this.client.query(
		{
			...mapMetricInputToQuery(this.context.inputs.nodes[i].node.source),
			limit: 1,
		},
		(metrics) => {
			if (metrics.length) {
				this.source = metrics[0];
			}
		},
		);
		this.colordata={from: source.name, color: nodes[j].node.colorSource };
		sankeydatas.push(colordata);
	}
	
	/*//Eventuele color conversion
		function componentToHex(c) {
			var hex = c.toString(16);
				return hex.length == 1 ? "0" + hex : hex;
			}

			function rgbToHex(r, g, b) {
				return "#" + componentToHex(r) + componentToHex(g) + componentToHex(b);
			}
			alert(rgbToHex(0, 51, 255)); // #0033ff*/
	
	for(i=0; i<nodes.length; i++){
		if(nodes[i] && nodes[i].node && metricHasQueryRequirements(this.context.inputs.nodes[i].node.source && metricHasQueryRequirements(this.context.inputs.nodes[i].node.target)){
			
			//Get last source metric
			this.client.query(
			{
				...mapMetricInputToQuery(this.context.inputs.nodes[i].node.source),
				limit: 1,
			},
			(metrics) => {
				if (metrics.length) {
					this.source = metrics[0];
				}
			},
			);
			
			//Get last target metric
			this.client.query(
			{
				...mapMetricInputToQuery(this.context.inputs.nodes[i].node.target),
				limit: 1,
			},
			(metrics) => {
				if (metrics.length) {
					this.target = metrics[0];
				}
			},
			);
			
			//Dit nog aanpassen wanneer nodig
			this.sankeydata = {from: source.name, to: target.name, value: target.value.rawValue, nodeColor: color };
			sankeydatas.push(sankeydata);
		}
	}
	
  },
  mounted() {
	this.$nextTick(() => {
		const chart = am4core.create(this.$refs.chart, am4charts.SankeyDiagram);
		chart.hiddenState.properties.opacity = 0; // this creates initial fade-in

		chart.data = sankeydatas;

		let hoverState = chart.links.template.states.create("hover");
		hoverState.properties.fillOpacity = 0.6;

		chart.dataFields.fromName = "from";
		chart.dataFields.toName = "to";
		chart.dataFields.value = "value";
		chart.dataFields.color = "nodeColor";

		// for right-most label to fit
		chart.paddingRight = 30;

		// make nodes draggable
		var nodeTemplate = chart.nodes.template;
		nodeTemplate.width = 20;
		nodeTemplate.clickable = true;
		nodeTemplate.draggable = false;

		var linkTemplate = chart.links.template;
		linkTemplate.propertyFields.fillOpacity = 0.5;
		linkTemplate.colorMode = "gradient";
	});
  },
  destroyed() {
    this.client.destroy();
  },
};
</script>

<style lang="scss" scoped>
@import '@/shared/styles/card';

.card .card-content {
  height: 100%;

  &.has-header {
    height: calc(100% - 40px);
  }

  .static {
    height: 100%;
    display: flex;
    flex-direction: row;
    place-content: center;
    align-items: center;
    flex: 1;

    span {
      white-space: nowrap;
    }
  }

  .scaled {
    height: 100%;
    display: flex;
    justify-content: center;

    svg {
      width: 100%;

      text {
        font-size: 14px;
        fill: var(--body-color);
      }
    }
  }
}
</style>
