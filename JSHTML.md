
# Javascript-Controlled HTML

In the previous assignment, we made a nextjs website.

NextJS is an api built on top of the hugely popular [React](https://reactjs.org) library. If you are required to build a website in the future, you'll probably deeply consider using React.

However, React presents an unusual architecture. React is _declarative_ instead of _imperative_. You do not directly (ie. imperatively) control the html when using React. Instead, you declare what you want rendered and the engine will reconcile what you want with what is currently on the page.

# Chess Game

We will learn React's declarative model more in the future. However, in this assignment, we are going to see just how cumbersome the more common imperative model can get.

We'll do this by rendering a very stateful game: Chess!

## Make a New Page

We are going to make a chess game directly in html with javascript. It will look similar to [this example in Wikipedia](https://en.wikipedia.org/wiki/Chess_symbols_in_Unicode#Chessboard_using_Unicode).

First things first, we need to make a new page in the NextJS project from last assignment.

Create a new `chess.js` file in the `pages` folder. Give it this code to start:
```jsx
export default function ChessPage() {
  return <div>Testing New Page</div>;
}
```

Now running `npm run dev` in your terminal should let you visit your new page at http://localhost:3000/chess

We will need some css on our page, so add a `Chess.module.css` file to the `styles` folder with this code:
```css
.board {
    text-align: center;
    border-spacing: 0pt;
    border-width: 0pt 0pt 0pt 0pt;
    user-select: none;
}

.square {
    font-family: 'Arial Unicode MS';
    font-size: 24pt;
    width: 1.5em;
    height: 1.5em;
    cursor: default;
}

.white {
    background-color: lightgrey;
}

.black {
    background-color: darkgrey;
}
```

And we can import those styles into `chess.js` with a line at the top:
```js
import styles from '../styles/Chess.module.css';
```

## Create the Board

The purpose of this assignment is to feel the architectural consequences of an imperative api (or simply: to see why React is useful), so, we will not be using React.

Instead, we'll pass off control to an imperative function:
```jsx
import styles from '../styles/Chess.module.css';

export default function ChessPage() {
  // Tell React to put a div down that is
  // controlled by the makeGame function
  return <div ref={makeGame} />;
}

function makeGame(div) {
  div.innerText = 'content now controlled by makeGame';
}
```

From here we can put a table into the div:
```js

function makeGame(div) {
  // make a new html <table> to render chess
  const board = document.createElement('table');
  board.className = styles.board;
  fillInBoard(board);

  // put that table into the div we control
  div.appendChild(board);
}

function fillInBoard(board) {
  // TODO
}
```

If you refresh the page, you should see: nothing particularly interesting. The `<table>` html element is there but isn't rendering much.

We've implemented filling in the board for you. Copy this as the implementation of `fillInBoard`:
```js
// makes a chess board out of an html table
function fillInBoard(board) {
  const COLNAMES = " ABCDEFGH";

  const body = document.createElement('tbody');

  // make each row in the table
  for (let r = 8; r >= 1; r--) {
    const row = document.createElement('tr');

    // number each row
    const rowLabel = document.createElement('td');
    rowLabel.innerText = r.toString();
    row.appendChild(rowLabel);

    // add the board squares
    for (let c = 1; c <= 8; c++) {
      const colName = COLNAMES[c];

      const square = document.createElement('td');
      square.id = colName + r;

      // color alternating squares
      const color = (r + c) % 2 ? styles.white : styles.black;
      square.className = styles.square + ' ' + color;

      row.appendChild(square);
    }

    body.appendChild(row);
  }

  // put column numbers on the bottom
  const footer = document.createElement('tr');
  for (let c = 0; c <= 8; c++) {
    const label = document.createElement('td');
    label.innerText = COLNAMES[c];

    footer.appendChild(label);
  }

  body.appendChild(footer);

  board.appendChild(body);
}
```

Now if you refresh the page, you should see an empty chessboard!

If you look carefully at the code, you'll see two things:
1. Each square has it's html `id` value set to the square's name ("A1" through "H8")
2. The code is hard to read (this is in large part because it is imperative)

We will use #1 in the later code.

## Initializing Chess

Let's use a [publicly available](https://github.com/josefjadrny/js-chess-engine) javascript chess engine. In your project, run `npm install js-chess-engine` then put at the top of `chess.js`:
```js
import * as Chess from 'js-chess-engine';
```

We can use this chess engine now to lay out the individual chess pieces.

Expand `makeGame` to be:
```js
// maps js-chess-engine's codes to text chess pieces
const GLYPHS = {
  K: "♔", Q: "♕", R: "♖", B: "♗", N: "♘", P: "♙",
  k: "♚", q: "♛", r: "♜", b: "♝", n: "♞", p: "♟︎",
};

function makeGame(div) {
  // make a new html <table> to render chess
  const board = document.createElement('table');
  board.className = styles.board;
  fillInBoard(board);

  // put that table into the div we control
  div.appendChild(board);

  // make a new chess game
  const game = new Chess.Game();
  let gameState = game.exportJson();

  // loop through and update all the squares with a piece on them
  Object.keys(gameState.pieces).forEach(square => {
    // square will be "A1" through "H8"

    // get the html element representing that square
    const el = document.getElementById(square);

    // take that piece and put its corresponding glyph into the square
    const piece = gameState.pieces[square];
    el.innerText = GLYPHS[piece];
  });
}
```

If all is working well, you should now have a chess board with pieces on it.

## Events

The browser, and many other software environments, is event-driven. Instead of having a tight loop of code that is constantly checking to see if a button is pressed, we instead provide a function that the system will call when the button is pressed.

In our case, we can put an `onclick` handler on all of our chessboard squares:
```js
function makeGame(div) {
  // ... (snip! code edited out for clarity) ...

  // make an onClick function
  const onClick = event => {
    const square = event.target.id;
    console.log('clicked ' + square);
  }

  // put that onClick function on every square
  Array.from(
    board.getElementsByClassName(styles.square)
  ).forEach(el => {
    el.onclick = onClick;
  });
}
```

If you refresh and look at your browser's [dev console](https://balsamiq.com/support/faqs/browserconsole/), you should now be able to click on each square and see which one you clicked on as a printout.

## Maintaining State

We now come to the point where we would like to play our chess game. That requires storing the state of the UI's interactions.

For instance, what happens when a user clicks a square is stateful:
1. If they previously clicked a piece and the new square is a valid move, move the piece to that square
2. If they previously clicked a piece and the new square is invalid, do nothing
3. If they haven't selected a piece and there is a piece on that square (that can move), select that piece.

How and where your app maintains its state is an important architectural question. For our very barebones code, we will simply store things in variables.

Let's make it very simple:
```js
// ... inside makeGame ...

// either null or the actively selected square
let selected = null;

// make an onClick function
const onClick = event => {
  const square = event.target.id;
  console.log('clicked ' + square);

  // check to see if we are moving a piece
  if (selected && gameState.moves[selected].includes(square)) {
    // move the piece
    game.move(selected, square);
    gameState = game.exportJson();

    // update the text by clearing out the old square
    document.getElementById(selected).innerText = "";
    // and putting the piece on the new square
    document.getElementById(square).innerText = GLYPHS[gameState.pieces[square]];

    // reset the selection state to unselected
    selected = null;
  } else if (selected) {
    // they tried to move a piece to a random spot on the board
    return;
  } else if (gameState.moves[square]) {
    // clicked on a piece that can move,
    // set the selection to that piece
    selected = square;
  }
}
```

Now when you run it, you will find that you can play chess! (You have to play both sides).

## On Your Own

The chess engine that we have imported can also compute what the next move should be. You can ask it to compute a move by calling:
```js
const [movedFrom, movedTo] = Object.entries(game.aiMove())[0];
```

On your own, have the ai play against you (meaning, you should make it move after each of your moves). Make sure to update the game board accordingly.

Also, the game is quite hard to use when it does not highlight your options.

Add a class selector to your css file:
```css
.square.isMoveOption {
    cursor: pointer;
    background-color: green;
}
```

Now, when you select a piece, go through that pieces move options (ie. `gameState.moves[square].forEach(...)`) and add the `styles.isMoveOption` css class to them.

When you make a move, remove the `sytles.isMoveOption` class.

You can add/remove classes from an element with:
```js
const el = document.getElementById('id');
el.classList.add('example');
el.classList.remove('other_example');
```

That should leave you with a very playable chess game.

So the requirements to submit are:
1. A chess game where selecting a piece highlights the possible squares
2. Selecting one of those squares moves the piece
3. The computer AI moves after your move
4. Publicly available on a nextjs site (which will read from your github)

When you've got that, post your chess game link and your github url to canvas. Feel free to have fun with features and styling! For instance, a challenge (if you want it) is to use `setTimeout` to have the AI wait for a bit before playing.

## Oops!

Did you notice any bugs when playing? It turns out, the system will start rendering incorrectly when someone castles their king or takes a pawn "en passant."

This is because we imperatively implemented the move as:
```js
// update the text by clearing out the old square
document.getElementById(selected).innerText = "";
// and putting the piece on the new square
document.getElementById(square).innerText = GLYPHS[gameState.pieces[square]];
```

However, that code only works if the only boxes that change are the destination and the origin of the moving piece.

In chess, that is mostly true. However, castling will move _two_ pieces and en passant will remove a piece that is _not_ on the destination square. So how can this be fixed?

Well, there are two basic approaches:
1. Manually implement castling/en-passant detection and mitigation. For instance, if the king moves non-adjacently, then assume the castle has moved too.
2. After each move, set each square to the game state.

This is a pretty good example of where imperative UI code goes wrong. Namely: whenever a state change is complicated or unusual.

In the declarative context (which is more like option 2), you don't implement _how_ the UI should change (there could be lots of ways for it to do so). Instead, you say _what it should be_ (eg. the `gameState` variable) and the internal engine will figure out the details.

> Note: you do not need to fix these bugs to turn in the assignment.

















