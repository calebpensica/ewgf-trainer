# ewgf-trainer
Work in progress!
Made by a backend dev trying to practice "front end"
Can select which character's sound effects are used when a WGF (Wind God Fist) or EWGF (Electric Wind God Fist) is correctly input from the dropdown.

Current Limitations:
Perfect Electrics (Where neutral is skipped) are not yet accounted for
13 frame electrics (Where D is skipped) are not yet accounted for
The page still logs d + f and d + f + 2 separately when it occurs at the same "frame"
Movement + Electrics (Dash electrics, sidestep electrics, backdash electrics, etc) are not yet accounted for
The page currently only supports Player 1 side
No support for input mapping yet
No scores or electric counters of any sort implemented yet

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
