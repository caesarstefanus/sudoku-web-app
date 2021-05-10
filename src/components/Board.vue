<template>
  <div>
    <table class="board">
      <tbody>
      <tr v-for="i in ROW" :key="`row_${i}`">
        <td v-for="j in COL" :key="`col_${j}`" class="cell">
          <template v-if="grid[i][j].isEditable">
          <v-menu 
            offset-y
            close-on-content-click
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                @mouseover="highlightRelatedCells(i, j)"
                @mouseleave="resetHighlightedCells()"
                style="font-size: 1.2em"
                height="100%" 
                block
                elevation="0"
                :outlined="!isHighlighted[i][j]"
                :color="isHighlighted[i][j] ? '#CABFCA' : (isBadCell(i, j) ? 'red' : '')"

                v-bind="attrs"
                v-on="on"
              >
                {{ grid[i][j].value }}
              </v-btn>
            </template>
            <tr
              v-for="numrow in NUMROW" 
              :key="`row_${i}col_${j}numrow_${numrow}`"
            >
              <td
                v-for="numcol in NUMCOL"
                :key="`row_${i}col_${j}num_${convertNumRowColToValue(numrow, numcol)}`"
              >
                <v-btn
                  block
                  tile
                  depressed
                  color="#ffffff"

                  @click="assignCellValue(i, j, convertNumRowColToValue(numrow, numcol))"
                >
                  {{ convertNumRowColToValue(numrow, numcol) }}
                </v-btn>
              </td>
            </tr>
            <tr>
              <td colspan="3">
              <v-btn
                block
                tile
                depressed

                @click="assignCellValue(i, j, null)"
              >
              <v-icon>mdi-eraser</v-icon>
              </v-btn>
              </td>
            </tr>
          </v-menu>
          </template>
          <template v-else>
            <v-btn
              height="100%"
              block
              outlined
              disabled

              v-bind="attrs"
              v-on="on"
            >
              {{ grid[i][j].value }}
            </v-btn>
          </template>
        </td>
      </tr>
      </tbody>
    </table>
    <v-container>
      <v-row>
        <v-spacer></v-spacer>
        <v-col>
          <v-btn
            @click="restartGame()"
            block
          >
            Restart
          </v-btn>
        </v-col>
        <v-spacer></v-spacer>
      </v-row>
      <v-row>
        <v-spacer></v-spacer>
        <v-col>
          <v-btn
            @click="useSolver()"
            block
          >
            Solve me!
          </v-btn>
        </v-col>
        <v-spacer></v-spacer>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { ROW, COL, NUMROW, NUMCOL } from "../constants/globalConstants.js"
// import sampleBoard from "../constants/sampleBoard.json";
import empty9by9board from "../constants/empty9by9board.json";
import { cloneDeep } from "lodash";

export default {
  name: 'Board',
  props: {
    msg: String
  },
  data() {
    return {
      ROW: ROW,
      COL: COL,
      NUMROW: NUMROW,
      NUMCOL: NUMCOL,

      on: null,
      attrs: null,

      grid: cloneDeep(empty9by9board),

      highlightedRow: 0,
      highlightedCol: 0,

      badRows: new Set(),
      badCols: new Set(),
      badSubgrids: new Set(),
    }
  },
  computed: {
    isHighlighted() {
      let arr = [];
      for (let i = 0; i <= ROW; ++i) {
        arr.push([]);
        for (let j = 0; j <= COL; ++j) {
          if (i === this.highlightedRow || j === this.highlightedCol
            || this.sameSubgrid(i, j, this.highlightedRow, this.highlightedCol)) {
            arr[i].push(true);
          } else {
            arr[i].push(false);
          }
        }
      }
      return arr;
    }
  },
  watch: {
    grid: {
      deep: true,
      handler(newvalue, oldvalue) {
        console.log("grid", newvalue, oldvalue)
        this.checkGrid();
      }
    }
  },
  methods: {
    assignCellValue(row, col, value) {
      console.log(row, col, value)
      if (this.grid[row][col].isEditable) {
        this.grid[row][col].value = value;
      }
    },
    convertNumRowColToValue(row, col) {
      return (row - 1) * NUMCOL + col;
    },
    highlightRelatedCells(row, col) {
      console.log(row, col, "highlight...")
      this.highlightedRow = row;
      this.highlightedCol = col;
    },
    resetHighlightedCells() {
      console.log("reset highlights")
      this.highlightedRow = 0;
      this.highlightedCol = 0;
    },
    getSubgrid(row, col) {
      return Math.floor((row - 1) / 3) * 3 + Math.floor((col - 1) / 3);
    },
    sameSubgrid(row1, col1, row2, col2) {
      return this.getSubgrid(row1, col1) === this.getSubgrid(row2, col2);
    },
    checkRow(row) {
      let s = new Set()
      for (let i = 1; i <= COL; ++i) {
        if (this.grid[row][i].value) {
          if (s.has(this.grid[row][i].value)) {
            this.badRows.add(row);
            return;
          }
          s.add(this.grid[row][i].value)
        }
      }
    },
    checkCol(col) {
      let s = new Set()
      for (let i = 1; i <= ROW; ++i) {
        if (this.grid[i][col].value) {
          if (s.has(this.grid[i][col].value)) {
            this.badCols.add(col);
            return;
          }
          s.add(this.grid[i][col].value)
        }
      }
    },
    checkSubgrid(y, x) {
      let s = new Set()
      for (let i = y * 3 - 2; i <= y * 3; ++i) {
        for (let j = x * 3 - 2; j <= x * 3; ++j) {
          if (this.grid[i][j].value) {
            if (s.has(this.grid[i][j].value)) {
              this.badSubgrids.add(this.getSubgrid(i, j));
              return;
            }
            s.add(this.grid[i][j].value)
          }
        }
      }
    },
    checkGrid() {
      this.badRows = new Set()
      this.badCols = new Set()
      this.badSubgrids = new Set()

      for (let i = 1; i <= ROW; ++i) {
        this.checkRow(i)
      }
      for (let i = 1; i <= COL; ++i) {
        this.checkCol(i)
      }
      for (let i = 1; i <= Math.floor(ROW / 3); ++i) {
        for (let j = 1; j <= Math.floor(COL / 3); ++j) {
          this.checkSubgrid(i, j)
        }
      }
    },
    isBadCell(row, col) {
      return this.badRows.has(row) || this.badCols.has(col) || this.badSubgrids.has(this.getSubgrid(row, col))
    },
    restartGame() {
      for (let i = 0; i <= ROW; ++i) {
        for (let j = 0; j <= COL; ++j) {
          this.$set(this.grid[i][j], 'value', empty9by9board[i][j].value)
          this.$set(this.grid[i][j], 'isEditable', empty9by9board[i][j].isEditable)
          console.log(i, j, this.grid[i][j])
          console.log(empty9by9board[i][j].value)
        }
      }
      console.log("done???")
    },
    useSolver() {
      // TODO: create solver and use!
    },
    debug() {
      console.log(this.grid)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.board {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 60vh;
}

.cell {
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 50px;
}

.cellButton {
  transition-duration: 0.4s;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.cellButton:hover {
  background-color: #4CAF50; /* Green */
  color: white;
  box-shadow: 0 12px 16px 0 rgba(0,0,0,0.24), 0 17px 50px 0 rgba(0,0,0,0.19);
}

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
