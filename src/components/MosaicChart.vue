
<template>
 
        <div class="container-fluid">
          <div class="row">
            <div class="col-md-10">
              <div id="chart" ref="chartContainer"></div>
            </div>
            <div class="col-md-2">
              <div class="deep-drill">
                <h3>Deep-drill into:</h3>
                <ul class="list-group">
                  <li class="list-group-item">Opportunities</li>
                  <li class="list-group-item">Closed win</li>
                  <li class="list-group-item">Closed lost</li>
                </ul>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-md-11">
              <table class="table table-borderless">
                <thead>
                  <tr>
                    <th></th>
                    <th v-for="h in tableHeader" :key= "h" class="text-end">
                      {{ h }}
                    </th>
                    <th></th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(values, key) in tableData" :key="key" class="text-end">
                    <td class="text-start">{{ getKey(key) }}</td>
                    <td v-for="(value, i) in values" :key="`${i}_${value}`">{{ key === 'numberOfDeals' ? value : `€${value}` }}</td>
                    <td class="cell--total">{{ key === 'numberOfDeals' ? summedValues(values, key): `€${summedValues(values, key)}` }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
    
</template>

<script>
import * as d3 from 'd3';
import _ from 'lodash';



export default {
  name: "mosaic",
  data: function () {
    return {
      tableHeader: [
        "Qualified",
        "Meeting Scheduled",
        "Offer",
        "Negotiation",
        "Closed win"
      ],
      tableData: {
        numberOfDeals: [
          4,3,5,5,5
        ],
        weightedDealValue: [
          20000, 30000,60000,  70000, 60000
        ],
        averageDealSize: [
          30000, 25000, 20000, 20000, 20000
        ]
      }
    }
  },
  computed: {},
  methods: {
    summedValues(values, key) {
      if(key === "averageDealSize") return _.mean(values);
      return _.sum(values);
    },
    getKey(key) {
      return _.upperFirst(_.lowerCase(key));
    }
  },
  mounted() {
    const chartContainer = this.$refs.chartContainer;
    const width = chartContainer.clientWidth;
    const height= 600;
    const margin = {
      top: 60,
      bottom: 20,
      left: 180
    };
    const fillColors= [ "#f0ac12", "#a6a6a6"];

    d3.json("./assets/dummy.json")
      .then(data => {
        window.d3 = d3;
        const locale = d3.formatLocale({
          decimal: '.',
          thousands: ',',
          grouping: [3],
          currency: ['€', '']
        })
        const format = locale.format("$,");
        const fillColor = d => fillColors[d] || "#3d3c3c";
        const strokeColor = d => d>1 ? "#fff" : "#000";
        let xIndex;
        let counter = 0;

        const treemap = data => d3.treemap()
              .round(true)
              .tile(d3.treemapSliceDice)
              .size([
                width - margin.left,
                height - margin.top - margin.bottom
              ])(
                d3.hierarchy(d3.group(data, d => d.type, d => d.subtype))
                    .sum(d => d.value)
                    .sort((a,b) => {
                      const val = a.depth === 2 ? a.value - b.value : b.value - a.value;
                      return val;

                    })

              ).each( d => {
                if(d.x0 === xIndex) {
                  counter++
                } else {
                  xIndex = d.x0;
                  counter = 0
                }
                d.i = counter;
                d.x0 += margin.left;
                d.x1 += margin.left;
                d.y0 += margin.top;
                d.y1 += margin.top;
              })

              const root = treemap(data)

              const svg = d3.create('svg')
                    .attr('viewBox', [0,0, width, height])

              const node = svg.selectAll('g')
                              .data(root.descendants())
                              .join('g')
                              .attr('transform', d => `translate(${d.x0}, ${d.y0})`)

              const column = node.filter(d => d.length === 1);

              column.append('text')
              .attr('x', d => {
                return (d.x1 - d.x0) / 2
              })
              .attr('dy', '-0.8em')
              .attr('fill', ' white')
              .style('text-anchor', 'middle')
              .text(d => format(d.value));

              const cell = node.filter(d => d.depth === 2);

               cell.append('rect')
                .attr('fill', d => fillColor(d.parent.children.length - (d.i + 1)))
                .attr('width', d => d.x1 - d.x0)
                .attr('height', d => d.y1 - d.y0)
                .attr('stroke', d => strokeColor(d.parent.children.length - (d.i + 1)));


              cell.append('text')
              .attr('x', d => (d.x1 - d.x0)/2)
              .attr('y', d => (d.y1 - d.y0)/2)
              .text(d=> d.data[0])
              .attr('fill', d => strokeColor(d.parent.children.length - (d.i + 1)))
              .style('text-anchor', 'middle')

              var yScale = d3.scaleLinear()
              .range([0, height - margin.top - margin.bottom]);

              var ytick = svg.selectAll('.y')
                .data(yScale.ticks(5))
                .enter()
                .append('g')
                .attr('class', 'y')
                .attr('transform', d => 'translate(' + margin.left +',' + (yScale(1 - d) + margin.top) + ')');
    
               ytick.append('text')
                .attr('x', -8)
                .attr('text-anchor', 'end')
                .attr('dy', '.35em')
                .attr('fill', 'white')
                .text(d3.format('.0%'));
              
              d3.select("#chart").append(() => svg.node())
            
      })

  }

}

</script>


<style>

.table {
  color: white;
}

.cell--total {
  font-weight: 600;
  color: #f0ac12;
}

div.deep-drill {
  margin-top: 20px;
}

.deep-drill > h3 {
  border-bottom: 3px dotted #f0ac12;
}

.list-group-item {
  background-color: #323236;
  color: white;
  border: 2px dashed white !important;
  margin: 5px;
  padding: 12px;
}

</style>