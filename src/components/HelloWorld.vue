<template>
  <body>
    <div>{{err}}</div>
    <div id="grid"></div>
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
      err: '',
      ox: -1,
      oy: -1,
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
      var playerRand = Math.floor(Math.random() * 10);
      var exitRand = Math.floor(Math.random() * 10);
      var player = false;
      var exit = false;

      // iterate for rows 
      for (var row = 0; row < 10; row++) {
        data.push( new Array() );
    
        // iterate for cells/columns inside rows
        for (var column = 0; column < 10; column++) {
          var rand = Math.floor(Math.random() * 10);
          var obstacle = false;
          if (rand < 3) obstacle = true;
          if (column == 0 && row == playerRand) {
            console.log(column, row)
            player = true;
          }
          else player = false;
          if (column == 9 && row == exitRand) {
            console.log(column, row)
            exit = true;
          }
          else exit = false;

          data[row].push({
            x: xpos,
            y: ypos,
            width: width,
            height: height,
            click: click,
            obstacle: obstacle,
            player: player,
            exit: exit,
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
    },

    check(direction) { 
      var result = true;

      this.column.each((d) => {
        if (direction == 'left') {
          if (this.ox == 1) result = false;
          if ((this.ox - 50 == d.x) && d.obstacle && this.oy == d.y) {
            result = false;
          }
        }
        else if (direction == 'right') {
          if (this.ox == 451) result = false;
          if ((this.ox + 50 == d.x) && d.obstacle && this.oy == d.y) {
            result = false;
          }
        }
        else if (direction == 'up') {
          if (this.oy == 1) result = false;
          if ((this.oy - 50 == d.y) && d.obstacle && this.ox == d.x) {
            result = false;
          }
        }
        else if (direction == 'down') {
          if (this.oy == 451) result = false;
          if ((this.oy + 50 == d.y) && d.obstacle && this.ox == d.x) {
            result = false;
          }
        }
      });
      return result;
    },

    moveRight() {
      this.getPlayerLoc();
      if (this.check('right')) {
        this.column
          .each((d) => {
            if (this.ox == d.x && this.oy == d.y)
              d.player = false;

            else if ((this.ox + 50) == d.x && this.oy == d.y)
              d.player = true;
          });
          this.update();
      }
      else {
        this.err = 'Cannot move right';
      }
    },

    moveLeft() {
      this.getPlayerLoc();
      if (this.check('left')) {
        this.column
          .each((d) => {
            if (this.ox == d.x && this.oy == d.y)
              d.player = false;

            else if ((this.ox - 50) == d.x && this.oy == d.y)
              d.player = true;
          });
          this.update();
      }
      else {
        this.err = 'Cannot move left';
      }
    },

    moveUp() {
      this.getPlayerLoc();
      if (this.check('up')) {
        this.column
          .each((d) => {
            if (this.ox == d.x && this.oy == d.y)
              d.player = false;

            else if (this.ox == d.x && (this.oy - 50) == d.y)
              d.player = true;
          });
          this.update();
      }
      else {
        this.err = 'Cannot move up';
      }
    },

    moveDown() {
      this.getPlayerLoc();
      if (this.check('down')) {
        this.column
          .each((d) => {
            if (this.ox == d.x && this.oy == d.y)
              d.player = false;

            else if (this.ox == d.x && (this.oy + 50) == d.y)
              d.player = true;
          });
          this.update();
      }
      else {
        this.err = 'Cannot move down';
      }
    },

    getPlayerLoc() {
      this.column.each((d) => {
        if (d.player) {
          this.ox = d.x;
          this.oy = d.y;
        }
      });
    },

    update() {
      this.column
        .attr("xlink:href", (d) => {
          if (d.player) { return "https://i.imgur.com/2BgE0Gt.png" }
          else if (d.exit) { return "https://i.imgur.com/X0SIYn3.png" }
          else if (d.obstacle) { return "https://i.imgur.com/9z6Yclr.png" }
          else { return "https://i.imgur.com/y4lTohA.png" }
        });
    }
  },
  mounted() {
    let recaptchaScript = document.createElement('script')
    recaptchaScript.setAttribute('src', 'https://d3js.org/d3.v4.min.js')
    document.head.appendChild(recaptchaScript)

    setTimeout(()=> {this.gridData = this.gridDataMethod();

    this.grid = d3.select("#grid")
      .append("svg")
      .attr("width","500px")
      .attr("height","500px");

    this.row = this.grid.selectAll(".row")
      .data(this.gridData)
      .enter().append("g")
      .attr("class", "row");

    this.column = this.row.selectAll(".square")
      .data(function(d) { return d; })
      //.enter().append("rect")
      .enter().append("svg:image")
      //.attr("class","square")
      .attr("xlink:href", (d) => {
        if (d.player) { return "https://i.imgur.com/2BgE0Gt.png" }
        else if (d.exit) { return "https://i.imgur.com/X0SIYn3.png" }
        else if (d.obstacle) { return "https://i.imgur.com/9z6Yclr.png" }
        else { return "https://i.imgur.com/y4lTohA.png" }
      })
      .attr("x", function(d) { return d.x; })
      .attr("y", function(d) { return d.y; })
      .attr("width", function(d) { return d.width; })
      .attr("height", function(d) { return d.height; 
      });
      
    }, 300)
    
    setTimeout(() => { this.moveRight() }, 1500)
    setTimeout(() => { this.moveDown() }, 3000)
    setTimeout(() => { this.moveLeft() }, 4500)
    setTimeout(() => { this.moveUp() }, 6000)
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
