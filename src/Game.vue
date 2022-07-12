<script setup lang="ts">
import { onUnmounted } from 'vue'
import { getWordOfTheDay, getRandomWord, allWords } from './words'
import Keyboard from './Keyboard.vue'
import { LetterState } from './types'
import {baseURL} from "./utils/config";

// dogeUI
const dogeUI = baseURL + "/doge.gif";
// dogeUrl
const dogeSite = "https://dogedoge.site";

const onKeyup = (e: KeyboardEvent) => onKey(e.key)

window.addEventListener('keyup', onKeyup)

onUnmounted(() => {
  window.removeEventListener('keyup', onKeyup)
})

// 获得题目
let answer = getWordOfTheDay()
// 二维数组答题板， [字母， 状态]
let board = $ref(
    Array.from({ length: 6 }, () =>
        Array.from({ length: 5 }, () => ({
          letter: '',
          state: LetterState.INITIAL
        }))
    )
)
// 当前行索引
let currentRowIndex = $ref(0)
// 当前行
let currentRow = $computed(() => board[currentRowIndex])
// 提示消息
let message = $ref('')
let grid = $ref('')
// 抖动行
let shakeRowIndex = $ref(-1)
let success = $ref(false)
// Handle keyboard input.
let allowInput = $ref(true)
// 单词&状态键盘
let letterStates = $ref({})

function init() {
  // 二维数组答题板， [字母， 状态]
  board = Array.from({ length: 6 }, () =>
      Array.from({ length: 5 }, () => ({
        letter: '',
        state: LetterState.INITIAL
      }))
  )

  // 当前行索引
  currentRowIndex = 0
  // 当前行
  currentRow = board[currentRowIndex]

  // 提示消息
  message = ''
  grid = ''
  // 抖动行
  shakeRowIndex = -1
  success = false

  // Handle keyboard input.
  allowInput = true
  // 单词&状态键盘
  letterStates = {}
}

// 初始化
init()

function onKey(key: string) {
  if (!allowInput) return
  if (/^[a-zA-Z]$/.test(key)) {
    fillTile(key.toLowerCase())
  } else if (key === 'Backspace') {
    clearTile()
  } else if (key === 'Enter') {
    completeRow()
  }
}

function fillTile(letter: string) {
  for (const tile of currentRow) {
    if (!tile.letter) {
      tile.letter = letter
      break
    }
  }
}

function clearTile() {
  for (const tile of [...currentRow].reverse()) {
    if (tile.letter) {
      tile.letter = ''
      break
    }
  }
}

function completeRow() {
  if (currentRow.every((tile) => tile.letter)) {
    const guess = currentRow.map((tile) => tile.letter).join('')
    // 输入的单词非法
    if (!allWords.includes(guess) && guess !== answer) {
      shake()
      showMessage(`Not in word list`)
      return
    }

    const answerLetters: (string | null)[] = answer.split('')
    // 先标记正确的单词
    currentRow.forEach((tile, i) => {
      if (answerLetters[i] === tile.letter) {
        tile.state = letterStates[tile.letter] = LetterState.CORRECT
        answerLetters[i] = null
      }
    })
    // 标记包含但错位的单词
    currentRow.forEach((tile) => {
      if (!tile.state && answerLetters.includes(tile.letter)) {
        tile.state = LetterState.PRESENT
        answerLetters[answerLetters.indexOf(tile.letter)] = null
        // 键盘状态修改
        if (!letterStates[tile.letter]) {
          letterStates[tile.letter] = LetterState.PRESENT
        }
      }
    })
    // 标记输入后不存在的单词
    currentRow.forEach((tile) => {
      if (!tile.state) {
        tile.state = LetterState.ABSENT
        if (!letterStates[tile.letter]) {
          letterStates[tile.letter] = LetterState.ABSENT
        }
      }
    })

    allowInput = false
    // 猜对了
    if (currentRow.every((tile) => tile.state === LetterState.CORRECT)) {
      setTimeout(() => {
        grid = genResultGrid()
        showMessage(
          ['Genius', 'Magnificent', 'Impressive', 'Splendid', 'Great', 'Phew'][
            currentRowIndex
          ],
          -1
        )
        success = true
      }, 1600)
    // 继续答题
    } else if (currentRowIndex < board.length - 1) {
      currentRowIndex++
      setTimeout(() => {
        allowInput = true
      }, 1600)
    } else {
      // 机会用尽
      setTimeout(() => {
        showMessage(answer.toUpperCase(), -1)
      }, 1600)
    }
  } else {
    shake()
    showMessage('Not enough letters')
  }
}

function showMessage(msg: string, time = 1000) {
  message = msg
  if (time > 0) {
    setTimeout(() => {
      message = ''
    }, time)
  }
}

function shake() {
  shakeRowIndex = currentRowIndex
  setTimeout(() => {
    shakeRowIndex = -1
  }, 1000)
}

const icons = {
  [LetterState.CORRECT]: '🟩',
  [LetterState.PRESENT]: '🟨',
  [LetterState.ABSENT]: '⬜',
  [LetterState.INITIAL]: null
}

function genResultGrid() {
  // 拼出成绩板
  return board
    .slice(0, currentRowIndex + 1)
    .map((row) => {
      return row.map((tile) => icons[tile.state]).join('')
    })
    .join('\r\n')
}

function shareGrid() {
  // 复制分享成绩
  var result = 'dogeorldle🐶 '+ (currentRowIndex + 1) + '/6 \r\n'
  // 动态创建 textarea 标签
  const textarea = document.createElement('textarea')
  // 将该 textarea 设为 readonly 防止 iOS 下自动唤起键盘，同时将 textarea 移出可视区域
  textarea.readOnly = true
  textarea.style.position = 'absolute'
  textarea.style.left = '-9999px'
  // 将要 copy 的值赋给 textarea 标签的 value 属性
  // 赋值给innerText也可以,但是识别不了\r\n的换行符，赋值给value属性就可以
  textarea.value = result + grid
  // 将 textarea 插入到 body 中
  document.body.appendChild(textarea)
  // 选中值并复制
  textarea.select()
  textarea.setSelectionRange(0, textarea.value.length)
  document.execCommand('Copy')
  document.body.removeChild(textarea)
  alert('copy to clipboard!')
}

function randomWordle() {
  answer = getRandomWord()
  init()
  alert('this function is for my lover ❤️ RuiXue \n Just enjoy it 😉')
}
</script>

<template>
  <Transition>
    <div class="message" v-if="message">
      {{ message }}
      <pre v-if="grid">{{ grid }}</pre>
      <button @click="shareGrid" v-if="grid">share :)</button><br/><br/>
      <span v-if="grid">play it at random: </span><button @click="randomWordle" v-if="grid">🔀</button>
    </div>
  </Transition>
  <header class="game-header">
    <div class="game-site"><a :href="dogeSite">doge小站
      </a></div>
    <div class="game-title">D<img :src="dogeUI" style="width: 30px; margin: auto"/>geordle</div>
    <div class="game-icon">
      <svg xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24" class="game-icon" data-testid="icon-statistics">
        <path fill="var(--color-tone-1)" d="M16,11V3H8v6H2v12h20V11H16z M10,5h4v14h-4V5z M4,11h4v8H4V11z M20,19h-4v-6h4V19z"></path>
      </svg>
    </div>
  </header>
  <div id="board-container">
    <div id="board">
      <div
        v-for="(row, index) in board"
        :class="[
          'row',
          shakeRowIndex === index && 'shake',
          success && currentRowIndex === index && 'jump'
        ]"
      >
        <div
          v-for="(tile, index) in row"
          :class="['tile', tile.letter && 'filled', tile.state && 'revealed']"
        >
          <div class="front" :style="{ transitionDelay: `${index * 300}ms` }">
            {{ tile.letter }}
          </div>
          <div
            :class="['back', tile.state]"
            :style="{
              transitionDelay: `${index * 300}ms`,
              animationDelay: `${index * 100}ms`
            }"
          >
            {{ tile.letter }}
          </div>
        </div>
      </div>
    </div>
  </div>
  <Keyboard @key="onKey" :letter-states="letterStates" />
</template>

<style scoped>
.game-header{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  flex-wrap: nowrap;
  padding: 0 16px;
  height: 50px;
  color: black;
  border-bottom: 1px solid #3a3a3c;
}
.game-title{
  font-weight: 700;
  font-size: 37px;
  line-height: 100%;
  letter-spacing: 0.01em;
  text-align: center;
  left: 0;
  right: 0;
  pointer-events: none;
  position: relative;
}
.game-icon{
  display: flex;
  width: 70px;
  justify-content: flex-end;
}
#board-container{
  display: flex;
  justify-content: center;
  align-items: center;
  flex-grow: 1;
  overflow: hidden;
}
#board {
  display: grid;
  grid-template-rows: repeat(6, 1fr);
  grid-gap: 5px;
  padding: 10px;
  box-sizing: border-box;
  --height: min(420px, calc(var(--vh, 100vh) - 310px));
  height: var(--height);
  width: min(350px, calc(var(--height) / 6 * 5));
  margin: 0px auto;
}
.message {
  position: absolute;
  left: 50%;
  top: 35%;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.85);
  padding: 16px 20px;
  z-index: 2;
  border-radius: 4px;
  transform: translateX(-50%);
  transition: opacity 0.3s ease-out;
  font-weight: 600;
}
.message.v-leave-to {
  opacity: 0;
}
.row {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-gap: 5px;
}
.tile {
  width: 100%;
  font-size: 2rem;
  line-height: 2rem;
  font-weight: bold;
  vertical-align: middle;
  text-transform: uppercase;
  user-select: none;
  position: relative;
}
.tile.filled {
  animation: zoom 0.2s;
}
.tile .front,
.tile .back {
  box-sizing: border-box;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}
.tile .front {
  border: 2px solid #d3d6da;
}
.tile.filled .front {
  border-color: #999;
}
.tile .back {
  transform: rotateX(180deg);
}
.tile.revealed .front {
  transform: rotateX(180deg);
}
.tile.revealed .back {
  transform: rotateX(0deg);
}

@keyframes zoom {
  0% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.shake {
  animation: shake 0.5s;
}

@keyframes shake {
  0% {
    transform: translate(1px);
  }
  10% {
    transform: translate(-2px);
  }
  20% {
    transform: translate(2px);
  }
  30% {
    transform: translate(-2px);
  }
  40% {
    transform: translate(2px);
  }
  50% {
    transform: translate(-2px);
  }
  60% {
    transform: translate(2px);
  }
  70% {
    transform: translate(-2px);
  }
  80% {
    transform: translate(2px);
  }
  90% {
    transform: translate(-2px);
  }
  100% {
    transform: translate(1px);
  }
}

.jump .tile .back {
  animation: jump 0.5s;
}

@keyframes jump {
  0% {
    transform: translateY(0px);
  }
  20% {
    transform: translateY(5px);
  }
  60% {
    transform: translateY(-25px);
  }
  90% {
    transform: translateY(3px);
  }
  100% {
    transform: translateY(0px);
  }
}

@media (max-height: 680px) {
  .tile {
    font-size: 3vh;
  }
}
</style>
