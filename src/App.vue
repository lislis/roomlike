<template>
    <header><h1>Roomlike</h1></header>
    <main>
        <div>
            <h2>Define items</h2>
            <p>List available items here.
                <br>One item per line.
                <br>Two items per line, separated by comma, mean they are dependend on each other and will both appear in the room, but individually.</p>

                <textarea :value="itemsRaw"
                          @change="update"></textarea>
                <button type="button">Parse input</button>
        </div>

        <div v-if="items.length > 0">
            <h2>Room Settings</h2>

            <p>Available items. Items on the same line will always be put in together.</p>
            <ul v-if="items.length">
                <li v-for="item in items">{{ item }}</li>
            </ul>

            <label>Number of items for solution (could be +1 depending on items)
                <input type="number" v-model="numSolutionItems">
            </label>
            <label>Max number of items in the room
                <input type="number" v-model="numMaxItems">
            </label>
            <label>Max number of tries
                <input type="number" v-model="numMaxTries">
            </label>
            <button type="button" @click="generateRoom">Generate room!</button>
        </div>

        <div v-if="solutionItems.size > 0">
            <h2>The Room</h2>
            <div class="room">
                <div v-for="x in roomItems" @click="tryItem" class="item">{{x}}</div>
            </div>

            <div v-if="!gameover && !win" class="blockies">
                <p>Solution:</p>
                <div v-for="x in solutionItems" class="blocky"></div>
            </div>

            <div v-if="tryHistory.length > 0">
                <p>Try history ({{ tryHistory.length}} / {{ numMaxTries }})</p>
                <ul>
                    <li v-for="t in tryHistory" >
                        <div v-for="item in t" class="item" :class="item.color">{{item.item}}</div>
                    </li>
                </ul>
            </div>

            <div v-if="currentItems.size > 0" class="blockies">
                <p>Current try: </p>
                <div v-for="x in currentItems" class="try-blocky">{{ x }}</div>
            </div>

            <div v-if="gameover">
                <h1>GAME OVER</h1>
                <p>Generate a new room to start over.</p>
            </div>
            <div v-if="win">
                <h1>YOU WIN</h1>
                <p>Good job.</p>
            </div>

        </div>
    </main>
</template>
<script>

 export default {
     name: "App",
     data() {
         return {
             itemsRaw: '',
             solutionItems: new Set(),
             currentItems: new Set(),
             roomItems: new Set(),
             tryHistory: [],
             numSolutionItems: 3,
             numMaxItems: 7,
             numMaxTries: 5,
             gameover: false,
             win: false
         }
     },
     methods: {
         update(evt) {
             this.itemsRaw = evt.target.value;
         },
         shuffleArray(arr) {
             arr.sort(() => Math.random() - 0.5);
         },
         generateRoom() {
             this.gameover = false;
             this.win = false;
             this.tryHistory = [];
             this.solutionItems = new Set();
             this.currentItems = new Set();
             let interrim = new Set();
             this.roomItems = new Set();

             if (this.numSolutionItems <= this.items.length) {
                 while (this.solutionItems.size < this.numSolutionItems) {
                     let newItem = this.items[Math.floor(Math.random()*this.items.length)];
                     if (newItem.length > 1) {
                         newItem.forEach(x => {
                             this.solutionItems.add(x);
                             interrim.add(x)
                         })
                     } else {
                         this.solutionItems.add(newItem[0])
                         interrim.add(newItem[0])
                     }
                 }
             } else {
                 console.log("not enough items")
             }

             if (this.numMaxItems <= this.items.length) {
                 while (interrim.size < this.numMaxItems) {
                     let newItem = this.items[Math.floor(Math.random()*this.items.length)];
                     if (newItem.length > 1) {
                         newItem.forEach(x => {
                             interrim.add(x)
                         })
                     } else {
                         interrim.add(newItem[0])
                     }
                 }

                 let arr = Array.from(interrim);
                 this.shuffleArray(arr);
                 this.roomItems = new Set(arr);
             } else {
                 console.log("not enough items")
             }
         },
         tryItem(evt) {
             if (!this.gameover && !this.win) {
                 this.currentItems.add(evt.target.textContent);
                 if (this.currentItems.size === this.solutionItems.size) {
                     this.checkSolution();
                 }
             }
         },
         checkSolution() {
             const current = Array.from(this.currentItems);
             const solution = Array.from(this.solutionItems);

             let outcome = current.map((c, i) => {
                 if (c === solution[i]) {
                     return { item: c, color: 'green'}
                 }
                 if (solution.indexOf(c) != -1) {
                     return { item: c, color: 'yellow'}
                 } else {
                     return { item: c, color: 'black'}
                 }
             });
             this.checkWin(outcome);
             this.tryHistory.push(outcome);
             this.currentItems = new Set();
             this.checkTries();
         },
         checkWin(arr) {
             if (arr.filter(x => x.color === 'green').length === this.solutionItems.size) {
                 this.win = true
             }
         },
         checkTries() {
             if (this.tryHistory.length === this.numMaxTries) {
                 this.gameover = true
             }
         }
     },
     computed: {
         items() {
             const items = this.itemsRaw
                               .split('\n')
                               .filter(x => x.length >= 1)
                               .map(x => x.split(','))
                               .map(x => x.filter(y => y !== ''));
             //console.log(items);
             return items;
         }
     }
 }
</script>
