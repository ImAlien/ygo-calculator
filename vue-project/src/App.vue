<template>
  <div id="main">
        <el-container>
        <el-header>
            <h3> ygo calculator</h3>
        </el-header>
        <el-main>
            <div class="main-header">
                <el-form
                :inline="true"
                :model="data"
                :rules="rules"
                >
                    <el-form-item label="dockSize" prop="dockSize">
                        <el-input v-model.number="data.dockSize"></el-input>
                    </el-form-item>
                    <el-form-item label="handSize" prop="handSize">
                        <el-input v-model.number="data.handSize"></el-input>
                    </el-form-item>
                </el-form>
                
            </div>
            <div class="main-card">
                <div v-for = "card,index in cards" :key="index">
                    <el-input v-model="card[0]" ></el-input>
                    <el-input v-model="card[1]" @input="changeConditionNumber(card)"></el-input>
                </div>  
                <div>
                    <el-button class="plus" type="info" @click="addCard"> + </el-button>
                    <el-button type="info" @click="removeCard"> - </el-button>
                    <el-button type="info" @click="clearCard"> C</el-button>
                </div>
            </div>
            <div class="main-content">
                <div v-for = "condition,index in conditions" :key ="index">
                    <div v-for = "item,index in condition" :key ="index">
                        <el-input v-model="item[0]" @input="bindnumber(item)"></el-input>
                        <el-input v-model="item[1]"></el-input>
                        <el-input v-model="item[2]"></el-input>
                    </div>        
                    <div>
                        <el-button class ="plus" type="info" @click="addItem(condition)"> + </el-button>
                        <el-button type="info" @click="removeItem(condition)"> - </el-button>
                        <el-button type="info" @click="clearItem(condition)"> C </el-button>
                    </div>    
                </div>
                <div>
                    <el-button class="plus" type="info" @click="addCondition"> + </el-button>
                    <el-button class="plus" type="info" @click="copyCondition">++</el-button>
                    <el-button type="info" @click="removeCondition"> - </el-button>
                    <el-button type="info" @click="clearCondition"> C </el-button>
                </div>
                
            </div>
            <div>
            </div>
        </el-main>
        <el-footer>
            <el-button type="info" @click="cal"><span class="compute-button">compute</span>  </el-button>
            <div><span>card:</span>{{cards_array}}</div>
            <div><span>possiblity:</span> {{result}}</div>
        </el-footer>
    </el-container>
    </div>
</template>

<script>


export default {
  el:"#main",
        // data() {
        //     return{
        //         dockSize:"20",
        //         handSize:"4"
        //     }
        // }
        data() {
            var intRange = (min, max) => (rule, value, callback) =>{
                if(typeof(value) === "string")
                    value = parseInt(value);
                console.log(typeof(value));
                if(!Number.isInteger(value)) {
                    callback("请输入一个整数");
                }
                else{
                    if(value < min || value > max){
                        callback(`范围是在${min}到${max}的整数`);
                    }
                    else callback();
                }
            }
            return{
                handSize:"4",
                cards:[["a",3],["b",3],["c",3],["d",3],["e",3],["f",3],["g",2]],
                vis:[],
                conditions:[[["",3,1]]],
                result:0,
                data:{
                    dockSize:'20',
                    handSize:'4',
                    cards:[["a",3],["b",3],["c",3],["d",3],["e",3],["f",3],["g",2]]
                },
                rules:{
                    dockSize:[
                        {validator:intRange(0,60), trigger:"blur"}
                    ],
                    handSize:[
                        {validator:intRange(4,6), trigger:"blur"}
                    ]
                }
            }
        },
        methods:{
            addCondition: function(){
                // console.log(this.cards_array);
                this.conditions.push([["",3,1]]);
            },
            removeCondition: function(){
                this.conditions.pop();
            },
            clearCondition: function(){
                this.conditions = [];
            },
            copyCondition(){
                let len = this.conditions.length;
                if(len >= 1)
                    this.conditions.push(JSON.parse(JSON.stringify(this.conditions[len- 1])));
            },
            addItem(condition){
                condition.push(["",3,1]);
            },
            removeItem(condition){
                condition.pop();
            },
            clearItem(condition){
                while(condition.length) condition.pop();
            },
            addCard: function(){
                this.cards.push(["",3]);
            },
            removeCard: function(){
                this.cards.pop();
            },
            clearCard: function(){
                this.cards=[];
            },
            cal:function(){
                let a = this.dockSize, b = this.handSize;
                let tot = this.fact(a)/this.fact(a-b);
                for(let i = 0; i < a; i ++) this.vis[i] = 0;
                let cur = [];
                let possible = this.dfs(0, cur);
                this.result = possible/tot;
            },
            fact(n){
                let ans = 1;
                for(let i = 1; i <= n; i ++) ans *= i;
                return ans;
            },
            dfs(pos,cur){
                let ans = 0;
                if(pos == this.handSize){
                    return this.check(cur);
                }
                for(let i = 0; i < this.dockSize; i ++){
                    if(this.vis[i] == 0){
                        this.vis[i] = 1;
                        cur.push(this.cards_array[i]);
                        ans +=this.dfs(pos+1, cur);
                        cur.pop();
                        this.vis[i] = 0;
                    }
                }
                return ans;
            },
            check(cur){
                for(const condition of this.conditions){
                    let flag = 1;
                    for(const item of condition){
                        let cnt = 0;
                        for (let x of cur){
                            if(x === item[0]) cnt ++;
                        }
                        if(cnt < item[2]){
                            flag = 0;break;
                        } 
                    }
                    if(flag) return 1;
                }
                return 0;
            },
            // 根据输入卡片绑定数量
            bindnumber(item) {
                let flag = false;
                for(let card of this.cards){
                    if(card[0] == item[0]){
                        flag = true;
                        this.$set(item, 1, card[1]);
                        break;
                    }
                }
                // console.log(item);
                if(flag == false) console.log("cards invalid");
            },
            //修改数量时条件中的数量也跟着改变
            changeConditionNumber(card){
                for(let condition of this.conditions){
                    for (let item of condition){
                        if(item[0] === card[0]){
                            this.$set(item, 1, card[1]);
                        }
                    }
                }
            }
        },
        computed:{
            cards_array: function(){
                let ans = [];
                for(let i in this.cards){
                    for(let j = 0; j < this.cards[i][1] ; j ++){
                        ans.push(this.cards[i][0]);
                    }
                }
                return ans;
            },
            dockSize(){
                let cnt = 0;
                for(let card of this.cards){
                    cnt += card[1]===""? 0: Number.parseInt(card[1]);
                }
                return cnt;
            }
        },
        //同步form中的data与Vue的data
        watch:{
            'data.dockSize'(val){
                this.dockSize = val;
                console.log(val);
            },
            'data.handSize'(val){
                this.handSize = val;
            },
            'data.cards'(val){
                this.cards = val;
                console.log(this.cards.toString());
            },
            dockSize(val){
                this.data.dockSize = val;
            }
        }
}
</script>

<style>
       html{
           width: 100%;
           height: 100%;
       }
       body{
           height: 100%;
           margin:0;
           color:rgba(255,255,255,0.7);
           font: 14px;
       }
       #main{
           width: 100%;
           height: 100%;
           margin: 0;
       }
       #main>.el-container{
           height: 100%;
           overflow: auto;
           background-color:#171717;
           display: flex;
       }
       #main>.el-container>.el-main{
           background-color:#171717;
           display: flex;
       }
       .el-header{
           background-color: #171717;
           display:flex;
           justify-content: center;
           margin-top:24px;
       }
       .el-footer{
           background-color: #171717;
       }
       .el-main{
           display: flex;
           flex-direction: column;
       }
       .main-header{
           display: flex;
           justify-content: center;
           margin-bottom: 24px;
       }   
       .main-header .el-input{
           width: 50%;
       }
       .main-card{
           display: flex;
           justify-content: center;
           flex-wrap: wrap;
           font: 12px;
           margin-bottom: 24px;
       }
       .main-card >div{
           display: flex;
           flex-direction: column;
           width: 100px;
           flex: 0 1 auto;
           margin:6px;
       }
       .main-card .el-button{
           margin-left: 0px;
           height: 13px;
       }
       .main-card  .el-input__inner{
           min-width: 30px;
           height: 30px;
           font-size: 12px;
           padding: 0px;
           text-align: center;
       }
       .main-content{
           display: flex;
           justify-content: center;
           flex-wrap: wrap;
       }
       .main-content >div{
           width: 300px;
           flex: 0 1 auto;
           display: flex;
           flex-direction: column;
           margin:8px;
       }
       .main-content >div div{
           flex: 1 1 auto;
           display: flex;
       }
       .main-content >div div .el-input{
           flex: 1 1 auto;
       }
       .main-content >.el-input{
           padding: 1px;
           height: 10px;
       }
       .main-content >div div .el-input .el-input__inner{
           width: 100%;
           height: 30px;
           font-size: 12px;
           padding: 0px;
           text-align: center;
       }
       .el-footer{
           display: flex;
           flex-direction: column;
           justify-content: center;
           align-items: center;
           margin-bottom: 24px;
       }
       .el-footer .el-button{
           width: 150px;
           height: 50px;
           margin-bottom: 16px;
       }
       .el-footer .el-button >span{
           height: 36px;
           font-size:28px;
       }
       .el-footer >div{
            margin-bottom:16px;
       }
       .el-input__inner{
           background-color: #171717;
           border-color:rgba(255,255,255,0.7);
           border: 1px solid;
       }
       .el-button{
           font-size: 6px;
           width: 20px;
           height: 15px;
           text-align: center;
           display: flex;
           margin:0px;
           justify-content:center;
           align-items: center;
           padding: 1px;
       }
       .el-button+.el-button{
           margin-left: 0px;
       }
       .plus{
           width: 25px;
           height: 15px;
       }
       /* el-form css */
       .el-form-item{
           margin: 0px;
       }
   </style>
