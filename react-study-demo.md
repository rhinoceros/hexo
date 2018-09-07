

``` shell
npm install -g create-react-app
create-react-app my-app
cd my-app 
rm -f src/*
```

### index.js

``` js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

function calculateWinner(squares,x,y) {
        let count_x=0;
        let count_y=0;
        let count_l=0;
        let count_r=0;
        for(let i=-2;i<=2;i++){
            if(x+i>=0  && x+i<=4  && squares[x][y]===squares[x+i][y])
            count_x++;
                if(count_x===3)
                {
                  return squares[x][y];
                }

            if(y+i>=0  && y+i<=4  && squares[x][y]===squares[x][y+i])
            {
                count_y++;
                if(count_y===3)
                {
                  return squares[x][y];
                }
            }
        
            if(x+i>=0 && y+i>=0  && x+i<=4 && y+i <=4 && squares[x][y]===squares[x+i][y+i])
            {
                count_l++;
                if(count_l===3)
                {
                  return squares[x][y];
                }
            }

            if(x+i>=0 && y-i>=0  && x+i<=4 && y-i <=4 && squares[x][y]===squares[x+i][y-i])
            {
                count_r++;
                if(count_r===3)
                {
                    return squares[x][y];
                }
            }
        }
    
  

    return null;
  }

function Square(props){
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
    renderSquare(i,j) {
      return (
        <Square
          x={i}
          y={j}
          value={this.props.squares[i][j]}
          onClick={() => this.props.onClick(i,j)}
        />
      );
    }
  
    render() {
      return (
        <div>
          <div className="board-row">
            {this.renderSquare(0,0)}
            {this.renderSquare(0,1)}
            {this.renderSquare(0,2)}
            {this.renderSquare(0,3)}
            {this.renderSquare(0,4)}
          </div>
          <div className="board-row">
            {this.renderSquare(1,0)}
            {this.renderSquare(1,1)}
            {this.renderSquare(1,2)}
            {this.renderSquare(1,3)}
            {this.renderSquare(1,4)}
          </div>
          <div className="board-row">
            {this.renderSquare(2,0)}
            {this.renderSquare(2,1)}
            {this.renderSquare(2,2)}
            {this.renderSquare(2,3)}
            {this.renderSquare(2,4)}
          </div>
          <div className="board-row">
            {this.renderSquare(3,0)}
            {this.renderSquare(3,1)}
            {this.renderSquare(3,2)}
            {this.renderSquare(3,3)}
            {this.renderSquare(3,4)}
        </div>
        <div className="board-row">
            {this.renderSquare(4,0)}
            {this.renderSquare(4,1)}
            {this.renderSquare(4,2)}
            {this.renderSquare(4,3)}
            {this.renderSquare(4,4)}
          </div>
        </div>
      );
    }
  }
  
  class Game extends React.Component {
      constructor(props){
          super(props);
          this.state={
              history: [{
                squares: Array(5).fill().map(() => Array(5).fill(null)),
                x: 0,
                y: 0,
                winner: null
              }],
              stepNumber: 0,
              xIsNext: true,
          };
      }
      handleClick(i,j) {
        console.log(this.state.history);
        const history=this.state.history.slice(0, this.state.stepNumber + 1);
        const current=history[history.length-1];
        const squares = current.squares.slice(0,5).map(s=>s.slice(0,5));
        
        if(current.winner||squares[i][j])
        {
            return;
        }
        squares[i][j] = this.state.xIsNext?'X':'0';
        const winner = calculateWinner(squares,i,j)
        this.setState({
            history: history.concat([{
                squares: squares,
                x: i,
                y: j,
                winner: winner,
            }]),
            stepNumber: history.length,
            xIsNext:!this.state.xIsNext,

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
        const history=this.state.history;
        const current = history[this.state.stepNumber];
        const winner=current.winner
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
        if(winner){
            status = "Winner: " + winner;
        }else{
            status = 'Next player: '+(this.state.xIsNext ? 'X' : 'O');
        }

      return (
        <div className="game">
          <div className="game-board">
            <Board
              squares={current.squares} 
              onClick={ (i,j) => this.handleClick(i,j)}
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
