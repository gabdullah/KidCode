<template>
  <body>
    <div id="container" class="row">
      <div id="grid" class="col-md-6"></div>

      <div id="terminal" class="col-md-6">
        <div class="row">
          <div id="terminal-log" class="col-sm-12">
            <ul>
              <li v-for="i in 8" :key="i">
                {{ commands[8-i] }} 
              </li>

              <!-- <li v-for="i in commands" :key="i">
                {{ commands[commands.length-i-1] }} 
              </li> -->
            </ul>
          </div>

          <div id="terminal-input" class="col-sm-12">
            <div class="input-group">
              <span style=";" class="form-control col-sm-1">></span>
              <input type="text" class="form-control col-sm-11" v-model="consoleInput" @keyup.enter="submitCommand(consoleInput)" @keyup.up="usePreviousCommands(1)" @keyup.down="usePreviousCommands(-1)" maxlength="15">    
            </div>
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
      err: '',
      ox: -1,
      oy: -1,
      commands: ['Welcome to KidCode! Type -h to get a list of commands.'],
      commandHistory: [],
      consoleInput: '',
      currentCommand: 0
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
          var enemyRand = Math.floor(Math.random() * 100);
          var enemyRand2 = Math.floor(Math.random() * 100);
          var enemy2 = false;
          var enemy = false;
          var obstacle = false;
          if (enemyRand2 < 5) {
            enemy2 = true;
            obstacle = false;
            enemy = false;
          }
          else if (enemyRand < 5) {
            enemy = true;
            obstacle = false;
            enemy2 = false;
          }
          else if (rand < 2) {
            obstacle = true;
            enemy = false;
            enemy2 = false;
          }
          
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
            enemy: enemy,
            enemy2: enemy2,
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
          if ((this.ox - 50 == d.x) && (d.obstacle || d.enemy || d.enemy2) && this.oy == d.y) {
            result = false;
          }
        }
        else if (direction == 'right') {
          if (this.ox == 451) result = false;
          if ((this.ox + 50 == d.x) && (d.obstacle || d.enemy || d.enemy2) && this.oy == d.y) {
            result = false;
          }
        }
        else if (direction == 'up') {
          if (this.oy == 1) result = false;
          if ((this.oy - 50 == d.y) && (d.obstacle || d.enemy || d.enemy2) && this.ox == d.x) {
            result = false;
          }
        }
        else if (direction == 'down') {
          if (this.oy == 451) result = false;
          if ((this.oy + 50 == d.y) && (d.obstacle || d.enemy || d.enemy2) && this.ox == d.x) {
            result = false;
          }
        }
      });

      if (result == false)
        this.commands.unshift('There is something blocking you from moving that way');

      this.enemyAttack();
      return result;
    },

    enemyAttack() {
      this.column.each((d) => {
      if ((this.ox + 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2)) ||
          (this.ox - 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2)) ||
          (this.oy + 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2)) ||
          (this.oy - 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2))) {
        var rand = Math.floor(Math.random() * 100);
        var num = Math.floor(Math.random() * 7);
        if (rand < 46) {
          this.commands.unshift('Mama bear (' + (Math.round(d.x/50)+1) + ', ' + (Math.round(d.y/50)+1) + ') slashes you for ' + num + ' damage!')
        }
        else {
          this.commands.unshift('Mama bear (' + (Math.round(d.x/50)+1) + ', ' + (Math.round(d.y/50)+1) + ') watches you impatiently')
        }
      }});

    },

    checkAttack(dir) {
      var result = false;

      // if ((this.ox + 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2)) ||
      //       (this.ox - 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2)) ||
      //       (this.oy + 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2)) ||
      //       (this.oy - 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2)))

      this.column.each((d) => {
        if (dir == 'left') {
          if (this.ox - 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2))
            result = true;
        }
        else if (dir == 'right') {
          if (this.ox + 50 == d.x && this.oy == d.y && (d.enemy || d.enemy2))
            result = true;
        }
        else if (dir == 'up') {
          if (this.oy - 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2))
            result = true;
        }
        else if (dir == 'left') {
          if (this.oy + 50 == d.y && this.ox == d.x && (d.enemy || d.enemy2))
            result = true;
        }
      });
        
      if (result == false)
        this.commands.unshift('There are no enemies in range');

      this.enemyAttack();
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
      // else {
      //   this.commands.unshift('Cannot move right');
      // }
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
      // else {
      //   this.commands.unshift('Cannot move left');
      // }
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
      // else {
      //   this.commands.unshift('Cannot move up');
      // }
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
      // else {
      //   this.commands.unshift('Cannot move down');
      // }
    },

    attackLeft() {
      this.getPlayerLoc();
      if (this.checkAttack('left')) {
        var rand = Math.floor(Math.random() * 10);
        this.commands.unshift('Attacked mama bear for ' + rand + ' damage!')
      }
    },

    attackRight() {
      this.getPlayerLoc();
      if (this.checkAttack('right')) {
        var rand = Math.floor(Math.random() * 10);
        this.commands.unshift('Attacked mama bear for ' + rand + ' damage!')
      }
    },

    attackUp() {
      this.getPlayerLoc();
      if (this.checkAttack('up')) {
        var rand = Math.floor(Math.random() * 10);
        this.commands.unshift('Attacked mama bear for ' + rand + ' damage!')
      }
    },

    attackDown() {
      this.getPlayerLoc();
      if (this.checkAttack('down')) {
        var rand = Math.floor(Math.random() * 10);
        this.commands.unshift('Attacked mama bear for ' + rand + ' damage!')
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
          else if (d.enemy) { return "https://i.imgur.com/RAJS4Df.png" }
          else if (d.enemy2) { return "https://i.imgur.com/88Rus2X.png" }
          else { return "https://i.imgur.com/y4lTohA.png" }
        });
      },

    submitCommand() { 
      if(this.consoleInput == '-h' ||this.consoleInput == '-help') {
        this.clear()
        this.commands.unshift('', 'attack -h | List attack commands', 'move -h | List move commands', 'Help Commands: ')
      }
      else if(this.consoleInput == 'move -h') {
        this.clear()
        this.commands.unshift('', 'moveLeft()', 'moveDown()', 'moveRight()', 'moveUp()', 'Move Commands: ')
      }
      else if(this.consoleInput == 'attack -h') {
        this.clear()
        this.commands.unshift('', 'attackLeft()', 'attackDown()', 'attackRight()', 'attackUp()', 'Attack Commands: ')
      }
      else if(this.consoleInput == 'clear') {
        this.clear()  
      } 
      else if(this.consoleInput == 'moveUp()') {
        this.moveUp()
        this.commands.unshift(this.consoleInput, '')
      }
      else if(this.consoleInput == 'moveRight()') {
        this.commands.unshift(this.consoleInput, '')
        this.moveRight()
      }
      else if(this.consoleInput == 'moveDown()') {
        this.commands.unshift(this.consoleInput, '')
        this.moveDown()
      }
      else if(this.consoleInput == 'moveLeft()') {
        this.commands.unshift(this.consoleInput, '')
        this.moveLeft()
      }
      else if(this.consoleInput == 'attackLeft()') {
        this.commands.unshift(this.consoleInput, '')
        this.attackLeft()
      }
      else if(this.consoleInput == 'attackRight()') {
        this.commands.unshift(this.consoleInput, '')
        this.attackRight()
      }
      else if(this.consoleInput == 'attackUp()') {
        this.commands.unshift(this.consoleInput, '')
        this.attackUp()
      }
      else if(this.consoleInput == 'attackDown()') {
        this.commands.unshift(this.consoleInput, '')
        this.attackDown()
      }
      else {
        this.commands.unshift('No command  ' + '"' + this.consoleInput + '"' + ' exists, please try another command or type -h to see a list of the commands')
      }
      this.commandHistory.push(this.consoleInput)
      console.log(this.commandHistory[0])
      this.consoleInput = '';
    },

    clear() {
      this.commands = []
    },

    usePreviousCommands(x) {
      this.currentCommand += x

      if(this.commandHistorycurrentCommand == this.commandHistory.length) 
        this.currentCommand = this.commandHistory.length - 1;
      else if(this.currentCommand < 0) {
        this.currentCommand = 0;
      }   

      this.consoleInput = this.commandHistory[this.currentCommand]
      console.log(this.consoleInput, this.currentCommand, x)
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
      .enter().append("svg:image")
      .attr("xlink:href", (d) => {
        if (d.player) { return "https://i.imgur.com/2BgE0Gt.png" }
        else if (d.exit) { return "https://i.imgur.com/X0SIYn3.png" }
        else if (d.obstacle) { return "https://i.imgur.com/9z6Yclr.png" }
        else if (d.enemy) { return "https://i.imgur.com/RAJS4Df.png" }
        else if (d.enemy2) { return "https://i.imgur.com/88Rus2X.png" }
        else { return "https://i.imgur.com/y4lTohA.png" }
      })
      .attr("x", function(d) { return d.x; })
      .attr("y", function(d) { return d.y; })
      .attr("width", function(d) { return d.width; })
      .attr("height", function(d) { return d.height; 
      });
      
    }, 300)
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #container {
    padding: 100px; 
    width: 101vw;
    height: 100vh;
    background-color: #33393B;  
  } 

  #grid {
    float: left; 
  }

  #terminal {
    float: right;
    background-color: #1E1E1E;
    width: 500px; 
    height: 400px;
    box-shadow: 5px 10px 18px #888888;
  }

  #terminal-log {
    height:320px;
    width: 100%;
    color: white; 
    overflow:auto;
  }

  #terminal-input {
    height: 80px;
    width: 100%; 
  }

  #terminal-textbox {
    max-width: 100%;
    padding: 10px;  
  }

  ul {
    padding-inline-start: 10px;
  }

  ul li {
    color: #f1f1f1;
    list-style: none; 
  }

  ul li::before {
    color: white;  
    content: "\003e"; 
    font-size: 1em; 
    padding-right: 1.1225em; 
    position: relative;
    top: 0em; 
}

  .form-control {
    height: 50px;
    margin-top: 15px;
    padding: 0px;
  }
  
  svg {
    box-shadow: 5px 10px 18px #888888;
    overflow: hidden;
    vertical-align: middle;
  }

  ::-webkit-scrollbar {
    width: 12px;
  }

  ::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(255,255,255,0.3); 
    border-radius: 10px;
  }

::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px  rgba(255,255,255,0.5);  
}
</style>
