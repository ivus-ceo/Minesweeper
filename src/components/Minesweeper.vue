<template>
  <main class="minesweeper">   
    <menu class="minesweeper__menu">
      <div class="menu__item">
        <span class="item__icon">💣</span>
        <small ref="bombs">{{ bombs }}</small>
      </div>

      <div class="menu__item">
        <span ref="timer">0:0</span>
      </div>

      <div class="menu__item">
        <span class="item__icon">🚩</span>
        <small ref="flags">{{ flags }}</small>
      </div>
    </menu>

    <div class="minesweeper__board" ref="board"></div>
  </main>
</template>

<script>
  import Button from '../components/inputs/Button.vue';

  export default {
    name: 'Minesweeper',

    components: {
      Button
    },
    
    props: {
      rows: {
        default: 10,
        type: Number
      },

      bombs: {
        default: 20,
        type: Number
      }
    },

    data() {
      return {
        flags: 0,
        
        timer: {
          seconds: 0,
          minutes: 0,
        },

        board: {
          width: null,
          height: null,
          columns: null,
          size: null,
          cells: []
        }
      }
    },

    methods: {
      drawTimer() {
        setInterval(() => {
          this.timer.seconds++;

          if (this.timer.seconds % 60 === 0) {
            this.timer.minutes++;
            this.timer.seconds = 0;
          }

          this.$refs.timer.innerHTML = `${ this.timer.minutes }:${ this.timer.seconds }`;
        }, 1000)
      },

      drawBoard() {
        this.board.width = parseInt(window.getComputedStyle(this.$refs.board).width) - parseInt(window.getComputedStyle(this.$refs.board).padding) * 2;
        this.board.height = parseInt(window.getComputedStyle(this.$refs.board).height) - parseInt(window.getComputedStyle(this.$refs.board).padding) * 4;

        this.board.size = Math.floor((this.board.width / this.rows) - 2);
        this.board.columns = Math.floor(this.board.height / this.board.size);

        console.log(this.rows, this.board.columns, this.board.size);

        /* Dynamic creation of cells */
        for (let i = 0; i < this.board.columns; i++) {
          for (let j = 0; j < this.rows; j++) {
            const cell = document.createElement('div')
            /* Add class associated with cell */
            cell.classList.add('cell')
            /* Set attribute with coordinate */
            cell.setAttribute('x', j)
            cell.setAttribute('y', i)
            /* Calculate size of cell */
            cell.setAttribute('style', `width: ${ this.board.size }px; height: ${ this.board.size }px`);
            /* Add function to cell */
            cell.onclick = (e) => {
              this.defuseCurrentCell(e);
            };
            /* Push new cell to the board */
            this.$refs.board.appendChild(cell)
            /* Push to cells array */
            this.board.cells.push(cell)
          }
        }

        /* Dynamic creation of bombs */
        for (let i = 0; i < this.bombs; i++) {
          this.board.cells[ Math.floor(Math.random() * this.board.cells.length) ].classList.add('bomb')
        }

        /* Count for bombs around cell */
        for (let i = 0; i < this.board.cells.length; i++) {    
          /* Get cells around current cell */ 
          const cells = this.getNeighbourCells(this.board.cells[i]);
          /* Counter for bombs found */ 
          let total = 0;
          /* Loop through around cells */ 
          cells.forEach((cell, index) => {
            if (cell !== null && cell.classList.contains('bomb') && index !== 4) total++
          });
          /* Set data attribute for total bombs found */ 
          if (!this.board.cells[i].classList.contains('bomb')) {
            this.board.cells[i].setAttribute('data', total);
          }
        }
      },

      defuseCurrentCell(e) {
        const cell = e.target;
        /* Check if cell is has a bomb */
        if (cell.classList.contains('bomb')) {
          this.explosion();
          return;
        }
        /* Check if cell is already checked */
        if (cell.classList.contains('checked')) return;
        /* Check if cell has bombs around it */
        if (parseInt(cell.getAttribute('data')) > 0) {
          /* Parse data of current cell */
          cell.innerHTML = cell.getAttribute('data');
        } else{
          /* Call recursion for cells */
          this.openNeighbourCells(cell);
        }
        /* Current cell will become checked */
        cell.classList.add('checked');
      },

      getNeighbourCells(cell) {
        /* Get coordinates */
        const x = parseInt(cell.getAttribute('x'));
        const y = parseInt(cell.getAttribute('y'));
        /* Empty array of cells */
        const cells = [];

        /* Middle / Current cell */
        cells[4] = document.querySelector(`.cell[x="${ x }"][y="${ y }"]`);

        /* Top */
        cells[1] = document.querySelector(`.cell[x="${ x }"][y="${ y - 1 }"]`);
        /* Left */
        cells[3] = document.querySelector(`.cell[x="${ x - 1 }"][y="${ y }"]`);
        /* Right */
        cells[5] = document.querySelector(`.cell[x="${ x + 1 }"][y="${ y }"]`);
        /* Bottom */
        cells[7] = document.querySelector(`.cell[x="${ x }"][y="${ y + 1 }"]`);

        /* Top Left */
        cells[0] = document.querySelector(`.cell[x="${ x - 1 }"][y="${ y - 1 }"]`);
        /* Top Right */
        cells[2] = document.querySelector(`.cell[x="${ x + 1 }"][y="${ y - 1 }"]`);
        /* Bottom Left */
        cells[6] = document.querySelector(`.cell[x="${ x - 1 }"][y="${ y + 1 }"]`);
        /* Bottom Right */
        cells[8] = document.querySelector(`.cell[x="${ x + 1 }"][y="${ y + 1 }"]`);

        /* Return cells */
        return cells;
      },

      openNeighbourCells(cell) {
        /* Get cells */
        const cells = this.getNeighbourCells(cell);

        /* Small timeout for interval */
        setTimeout(() => {
          /* Loop through neighbour cells */
          cells.forEach((cell, index) => {
            /* If current cell exists and doesn't contain class checked and has attribute data with 0 length */
            // if (cells[index] !== null && !cells[index].classList.contains('checked') && cells[index].getAttribute('data') !== null && parseInt(cells[index].getAttribute('data')) === 0) {
            if (cells[index] !== null && !cells[index].classList.contains('checked') && cells[index].getAttribute('data') !== null) {
              /* Add class of checked to current cell */
              cells[index].classList.add('checked');
              if (parseInt(cells[index].getAttribute('data')) > 0) {
                cells[index].innerHTML = cells[index].getAttribute('data');
              } else {
                /* Call recursion */
                this.openNeighbourCells(cells[index]);
              }
            }
          });
        }, 10);
      },

      explosion() {
        for (let i = 0; i < this.board.cells.length; i++) {
          this.board.cells[i].onclick = null;

          if (this.board.cells[i].classList.contains('bomb')) {
            this.board.cells[i].innerHTML = '💣';
          }
        }
      },

      /**
       * Subsidiary functions
       */
      getSizeOfCell() {
        
      }
    },

    mounted() {
      this.drawTimer();
      this.drawBoard();
    },
  }
</script>

<style lang="scss">
  main.minesweeper {
    display: flex;
    max-width: 400px;
    flex-direction: column;
    width: calc(100vw - 1rem);
    height: calc(100vh - 1rem);
    font-family: 'Nunito', sans-serif;

    menu.minesweeper__menu {
      display: flex;
      padding: 0 1rem;
      align-items: center;
      justify-content: space-between;

      .menu__item {
        display: flex;
        align-items: center;
        flex-direction: column;

        &:first-of-type, &:last-of-type {
          width: 42px;
          height: 42px;
          cursor: pointer;
        }
      }
    }

    div.minesweeper__board {
      flex: 1;
      padding: 1rem;
      display: flex;
      flex-wrap: wrap;

      .cell {
        margin: 1px;
        display: flex;
        color: white;
        cursor: pointer;
        align-items: center;
        border-radius: .4rem;
        justify-content: center;
        background-color: #d1d8e0;

        &:hover, &:focus {
          background-color: darken(#d1d8e0, 10);
        }

        &.bomb {
          // background-color: red;
        }

        &.checked {
          background-color: lighten(#d1d8e0, 12);
        }

        &.flag {
          background-color: hotpink;
        }

        &.cell[data="1"].checked {
          background-color: #2d98da;
        }

        &.cell[data="2"].checked {
          background-color: #20bf6b;
        }

        &.cell[data="3"].checked {
          background-color: #eb3b5a;
        }

        &.cell[data="4"].checked {
          background-color: #fa8231;
        }

        &.cell[data="5"].checked, 
        &.cell[data="6"].checked, 
        &.cell[data="7"].checked, 
        &.cell[data="8"].checked {
          background-color: #8854d0;
        }
      }
    }

    div.board {
      width: 100%;
      height: 100%;
      display: flex;
      flex-wrap: wrap;
      max-width: 400px;
      max-height: 400px;
      align-content: center;
      animation: 1s backInUp;
      justify-content: center;

      div.cell {
        margin: 1px;
        display: flex;
        color: white;
        align-items: center;
        border-radius: .4rem;
        justify-content: center;
        background-color: #d1d8e0;
      }

      div.bomb {
        background-color: orange;
      }

      div.checked {
        background-color: lighten(#d1d8e0, 12);
      }

      div.flag {
        background-color: hotpink;
      }
    }
  }
</style>