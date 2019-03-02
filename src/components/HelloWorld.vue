<template>
  <body>
    <div id="container" class="row">
      <div id="grid" class="col-md-6"></div>

      <div id="terminal" class="col-md-6">
        <div class="row">
          <div id="terminal-log" class="col-sm-12">
            

          </div>

          <div id="terminal-input" class="col-sm-12">
            

          </div>
        </div>
      </div>
    </div>
  </body>
</template>

<script>
export default {
  name: 'HelloWorld',
  data() {
    return {
      gridData: '',
      grid: '',
      row: '',
      column: '',
    }
  },
  props: {
    msg: String
  },
  methods: {
    gridDataMethod() {
      var data = new Array();
      var xpos = 1; //starting xpos and ypos at 1 so the stroke will show when we make the grid below
      var ypos = 1;
      var width = 50;
      var height = 50;
      var click = 0;
  
      // iterate for rows 
      for (var row = 0; row < 10; row++) {
        data.push( new Array() );
    
        // iterate for cells/columns inside rows
        for (var column = 0; column < 10; column++) {
          data[row].push({
            x: xpos,
            y: ypos,
            width: width,
            height: height,
            click: click
          })
          // increment the x position. I.e. move it over by 50 (width variable)
          xpos += width;
        }
        // reset the x position after a row is complete
        xpos = 1;
        // increment the y position for the next row. Move it down 50 (height variable)
        ypos += height; 
      }
      return data;
    }
  },
  mounted() {
    let recaptchaScript = document.createElement('script')
    recaptchaScript.setAttribute('src', 'https://d3js.org/d3.v4.min.js')
    document.head.appendChild(recaptchaScript)

    setTimeout(()=> {this.gridData = this.gridDataMethod();
      this.grid = d3.select("#grid")
        .append("svg")
        .attr("width","510px")
        .attr("height","510px");

      this.row = this.grid.selectAll(".row")
        .data(this.gridData)
        .enter().append("g")
        .attr("class", "row");

      this.column = this.row.selectAll(".square")
        .data(function(d) { return d; })
        .enter().append("rect")
        .attr("class","square")
        .attr("x", function(d) { return d.x; })
        .attr("y", function(d) { return d.y; })
        .attr("width", function(d) { return d.width; })
        .attr("height", function(d) { return d.height; })
        .style("fill", "#fff")
        .style("stroke", "#222")
        .on('click', function(d) {
          d.click ++;
          if ((d.click)%4 == 0 ) { d3.select(this).style("fill","#fff"); }
          if ((d.click)%4 == 1 ) { d3.select(this).style("fill","#2C93E8"); }
          if ((d.click)%4 == 2 ) { d3.select(this).style("fill","#F56C4E"); }
          if ((d.click)%4 == 3 ) { d3.select(this).style("fill","#838690"); }
          });
    }, 300)
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #grid {
    float: left; 
  }

  #container {
    padding: 100px; 
  } 

  #terminal {
    background-color: black;
    width: 500px; 
    height: 400px;
    float: right;
  }

  #terminal-log {
    background-color: blue;
    height:320px;
    width: 100% ; 
  }

  #terminal-input {
    background-color: red;
    height: 80px;
    width: 100%; 
    margin: 10px; 
  }
</style>
