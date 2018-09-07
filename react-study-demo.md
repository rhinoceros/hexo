---
title: react js
date: 2018-09-07 12:00
categories: Chanjet
tags: 
- react
- js
- tutuorial
---

### https://reactjs.org/tutorial/tutorial.html
### 3\*3 -> 5\*5


### create app

``` shell
npm install -g create-react-app
create-react-app my-app
cd my-app 
rm -f src/*
```

### index.js

# 有BUG

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

function calculateWinner(squares, x, y) {
  let count_x = 0;
  let count_y = 0;
  let count_l = 0;
  let count_r = 0;
  let pre_x = null;
  let pre_y = null;
  let pre_l = null;
  let pre_r = null;

  for (let i = -5; i <= 5; i++) {
    if (x + i >= 0 && x + i <= 15 && squares[x][y] === squares[x + i][y]) {
      if (pre_x) {
        if (pre_x == squares[x + i][y]) {
          count_x++;
          if (count_x === 5) {
            return squares[x][y];
          }
        } else {
          count_x = 0;
        }
      }
      else {
        pre_x = squares[x][y];
      }

    }

    if (y + i >= 0 && y + i <= 15 && squares[x][y] === squares[x][y + i]) {
      if (pre_y) {
        if (pre_y == squares[x][y + i]) {
          count_y++;
          if (count_y === 5) {
            return squares[x][y];
          }
        } else {
          count_y = 0;
        }
      }
      else {
        pre_y = squares[x][y];
      }
    }

    if (x + i >= 0 && y + i >= 0 && x + i <= 15 && y + i <= 15 && squares[x][y] === squares[x + i][y + i]) {

      if (pre_l) {
        if (pre_l == squares[x + i][y + i]) {
          count_l++;
          if (count_l === 5) {
            return squares[x][y];
          }
        }
        else {
          count_l = 0;
        }
      }
      else {
        pre_l = squares[x][y];
      }



    }

    if (x + i >= 0 && y - i >= 0 && x + i <= 15 && y - i <= 15 && squares[x][y] === squares[x + i][y - i]) {
      count_r++;
      if (count_r === 5) {
        return squares[x][y];
      }

      if (pre_r) {
        if (pre_r == squares[x + i][y - i]) {
          count_r++;
          if (count_r === 5) {
            return squares[x][y];
          }
        }
        else {
          count_r = 0;
        }
      }
      else {
        pre_r = squares[x][y];
      }
    }
  }



  return null;
}

function Square(props) {
  return (
    <button
      className="square"
      onClick={props.onClick}
      x={props.x}
      y={props.y}
    >
      {props.value}
    </button>
  );
}


class Board extends React.Component {
  renderSquare(i, j) {
    return (
      <Square
        x={i}
        y={j}
        value={this.props.squares[i][j]}
        onClick={() => this.props.onClick(i, j)}
      />
    );
  }

  render() {
    var squares_list = [];
    for (var m = 0; m <= 15; m++) {
      squares_list.push(
        <div className="board-row">
          {this.renderSquare(m, 0)}
          {this.renderSquare(m, 1)}
          {this.renderSquare(m, 2)}
          {this.renderSquare(m, 3)}
          {this.renderSquare(m, 5)}
          {this.renderSquare(m, 6)}
          {this.renderSquare(m, 7)}
          {this.renderSquare(m, 8)}
          {this.renderSquare(m, 9)}
          {this.renderSquare(m, 10)}
          {this.renderSquare(m, 11)}
          {this.renderSquare(m, 12)}
          {this.renderSquare(m, 13)}
          {this.renderSquare(m, 14)}
          {this.renderSquare(m, 15)}
        </div>
      )
    }



    return (<div>
      {squares_list}
    </div>)
  }
}

class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [{
        squares: Array(16).fill().map(() => Array(16).fill(null)),
        x: 0,
        y: 0,
        winner: null
      }],
      stepNumber: 0,
      xIsNext: true,
    };
  }
  handleClick(i, j) {
    console.log(this.state.history);
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice(0, 16).map(s => s.slice(0, 16));

    if (current.winner || squares[i][j]) {
      return;
    }
    squares[i][j] = this.state.xIsNext ? 'X' : '0';
    const winner = calculateWinner(squares, i, j)
    this.setState({
      history: history.concat([{
        squares: squares,
        x: i,
        y: j,
        winner: winner,
      }]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext,

    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0,
      x: 0,
      j: 0,
      winner: null,
    });
  }
  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = current.winner
    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li key={move} >
          <a href="#" onClick={() => this.jumpTo(move)}>{desc}</a>
        </li>
      );
    });
    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={(i, j) => this.handleClick(i, j)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
}

// ========================================

ReactDOM.render(
  <Game />,
  document.getElementById('root')
);

```

### index.css
``` css
body {
  font: 14px "Century Gothic", Futura, sans-serif;
  margin: 20px;
}

ol, ul {
  padding-left: 30px;
}

.board-row:after {
  clear: both;
  content: "";
  display: table;
}

.status {
  margin-bottom: 10px;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.square:focus {
  outline: none;
}

.kbd-navigation .square:focus {
  background: #ddd;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}

```
