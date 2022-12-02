<template>
    <header><h1>Roomlike</h1></header>
    <main>
        <div>
            <h2>Item pool</h2>
            <p>List available items here.
                <br>One item per line.
                <br>Two items per line, separated by comma, mean they are dependend on each other and will both appear in the room, but individually.</p>
                <textarea :value="itemsRaw"
                          @change="update"></textarea>
                <button type="button">Parse input</button>
        </div>

        <div>
            <h2>Available Items</h2>
            <ul v-if="items.length">
                <li v-for="item in items">{{ item }}</li>
            </ul>
            <button type="button" @click="generateRoom">Generate room!</button>
        </div>

        <div v-if="solutionItems.size > 0">
            <h2>Room generated</h2>
            <div class="blockies">
                <div v-for="x in solutionItems" class="blocky"></div>
            </div>

            <div v-if="tryHistory.length > 0">
                <p>try history</p>
                <ul>
                <li v-for="t in tryHistory" >
                    <div v-for="item in t" class="item" :class="item.color">{{item.item}}</div>
                </li>
                </ul>
            </div>

            <div v-if="currentItems.size > 0" class="blockies">
                <p>current try: </p>
                <div v-for="x in currentItems" class="">{{ x }}</div>
            </div>
            <div class="room">
                <div v-for="x in solutionItems" @click="tryItem" class="item">{{x}}</div>


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
             tryHistory: []
         }
     },
     methods: {
         update(evt) {
             this.itemsRaw = evt.target.value;
         },
         generateRoom() {
             this.tryHistory = [];
             this.solutionItems = new Set();
             this.currentItems = new Set();

             while (this.solutionItems.size <= 2) {
                 let newItem = this.items[Math.floor(Math.random()*this.items.length)];
                 if (newItem.length > 1) {
                     newItem.forEach(x => this.solutionItems.add(x))
                 } else {
                     this.solutionItems.add(newItem[0])
                 }

                 //this.solutionItems.add(newItem);
             }
         },
         tryItem(evt) {
             //console.log(evt.target.textContent)
             this.currentItems.add(evt.target.textContent);
             if (this.currentItems.size === this.solutionItems.size) {
                 this.checkSolution();
             }
         },
         checkSolution() {
             const current = Array.from(this.currentItems);
             const solution = Array.from(this.solutionItems);

             let outcome = current.map((c, i) => {
                 //debugger
                 if (c === solution[i]) {
                     return { item: c, color: 'green'}
                 }
                 if (solution.indexOf(c) != -1) {
                     return { item: c, color: 'yellow'}
                     //return 'yellow'
                 } else {
                     return { item: c, color: 'black'}
                     //                     return 'black'
                 }
             });
             this.tryHistory.push(outcome);
             this.currentItems = new Set();
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

<style>
 main {
     display: flex;
     flex-flow: row nowrap;
 }
 main > div {
     flex: 1 1 30%;
     padding: 0 2em;
     border-right: 2px currentColor dotted;
 }
 textarea {
     display: block;
 }
 .blocky {
     width: 1em;
     height: 1em;
     display: inline-block;
     margin-right: 0.2em;
     background-color: black;
 }

 .room {
     background-color: lightgrey;
     padding: 1em;
 }

 .item {
     display: inline-block;
     cursor: pointer;
     padding: 0.2em;
     margin-right: 0.2em;
     background-color: grey;
 }
 .item.green {
     background-color: green
 }
 .item.yellow {
     background-color: yellow
 }
</style>
