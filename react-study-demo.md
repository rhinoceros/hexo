
``` js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

var x=0;
var y=0;
var WINNER=""

function calculateWinner(squares) {
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
              }],
              stepNumber: 0,
              x: 0,
              y: 0,
              xIsNext: true,
          };
      }
      handleClick(i,j) {
        console.log(this.state.history);
        const history=this.state.history.slice(0, this.state.stepNumber + 1);
        const current=history[history.length-1];
        const squares = current.squares.slice(0,5).map(s=>s.slice(0,5));
        x=i;
        y=j;
        
        if(WINNER||calculateWinner(squares)||squares[i][j])
        {
            return;
        }
        squares[i][j] = this.state.xIsNext?'X':'0';

        

       
        this.setState({
            history: history.concat([{
                squares: squares,
            }]),
            stepNumber: history.length,
            xIsNext:!this.state.xIsNext,
            x: i,
            y: j,
        });
    }
    jumpTo(step) {
        this.setState({
          stepNumber: step,
          xIsNext: (step % 2) === 0,
          x: 0,
          j: 0,
        });
      }
    render() {
        const history=this.state.history;
        const current = history[this.state.stepNumber];
        const winner=calculateWinner(current.squares)
        

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
            WINNER=winner
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
