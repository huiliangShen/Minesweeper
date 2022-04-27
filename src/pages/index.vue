<script setup lang="ts">

interface IMimeItem {
  value: string | number
  flip: boolean
  mime: boolean
  color?: string
}
const MimeNumber = 10
const Edge = 10
const data = ref<Array<Array<IMimeItem | null>>>([])
let state = 0
const timer = ref(0)
const interval = ref()
// import { useUserStore } from '~/stores/user'

// const user = useUserStore()
// const name = $ref(user.savedName)

// const router = useRouter()
// const go = () => {
//   if (name)
//     router.push(`/hi/${encodeURIComponent(name)}`)
// }

// const { t } = useI18n()
/**
 * 洗牌算法
 * @param arr
 */
const shuffle = (arr: boolean[]) => {
  for (let i = arr.length - 1; i > 0; i--) {
    const randomIndex = Math.floor(Math.random() * (i + 1))
    const temp = arr[randomIndex]
    arr[randomIndex] = arr[i]
    arr[i] = temp
  }
}

const randomMine = (startX: number, startY: number) => {
  const arr = Array(Edge * Edge).fill(false)
  for (let i = 0; i < MimeNumber; i++)
    arr[i] = true
  shuffle(arr)
  const rows = []
  for (let i = 0; i < Edge; i++) {
    rows[i] = [] as IMimeItem[]
    for (let j = 0; j < Edge; j++) {
      if (i === startX && j === startY) {
        rows[i][j] = {
          value: '',
          flip: false,
          mime: false,
        }
      }
      else {
        const index = i * Edge + j
        rows[i][j] = {
          value: arr[index] ? '💣' : '',
          flip: false,
          mime: arr[index],
        }
      }
    }
  }
  return rows
}

const area = [
  [-1, -1], [-1, 0], [-1, 1],
  [0, -1], [0, 1],
  [1, -1], [1, 0], [1, 1],
]

const COLOR = [
  '',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-red-500',
]

const generationNumber = (arr: IMimeItem[][]) => {
  for (let i = 0; i < Edge; i++) {
    for (let j = 0; j < Edge; j++) {
      if (arr[i][j].mime)
        continue

      let sum = 0
      area.forEach(([x, y]) => {
        if (x + i >= 0 && x + i < Edge && y + j >= 0 && y + j < Edge)
          sum += Number(arr[x + i][y + j].mime)
      })

      arr[i][j].value = sum
      arr[i][j].color = COLOR[sum]
    }
  }
}

const create = () => {
  const d = Array.from({ length: Edge }, () => Array.from({ length: Edge }, () => null))
  data.value = d
}
/**
 * 清楚所有的状态
 */
const clearState = () => {
  interval.value && clearInterval(interval.value)
  interval.value = null
  timer.value = 0
  state = 0
}

/**
 * 生成炸弹和周边数字
 */
const initMime = (i: number, j: number) => {
  const arr = randomMine(i, j)
  generationNumber(arr)
  data.value = arr
}

const timerRun = () => {
  interval.value = setInterval(() => {
    timer.value++
  }, 1000)
}

const toggle = (i: number, j: number) => {
  if (state === 0) {
    initMime(i, j)
    timerRun()
    state++
  }
  else if (state === 2) {
    return
  }

  const d = data.value as IMimeItem[][]
  const map = new Map()
  if (d[i][j]?.flip)
    return

  d[i][j]!.flip = true

  if (d[i][j]?.mime) {
    setTimeout(() => {
      clearInterval(interval.value)
      alert('失败了')
      state++
    }, 50)
  }

  else if (!d[i][j]?.value) { dfs(d, i, j) }

  if (d.flat(2).filter(e => !e.flip).length === MimeNumber) {
    setTimeout(() => {
      clearInterval(interval.value)
      alert('congratulation!!!')
      state++
    }, 50)
  }

  function dfs(list: IMimeItem[][], _i: number, _j: number) {
    for (const [x, y] of area) {
      const x0 = x + _i; const y0 = y + _j
      if (x0 < 0 || x0 >= Edge || y0 < 0 || y0 >= Edge || map.has(`${x0}_${y0}`))
        continue
      const item = list[x0][y0]
      item.flip = true
      map.set(`${x0}_${y0}`, 1)
      if (!item.value)
        dfs(list, x0, y0)
    }
  }
}

const newGame = () => {
  create()
  clearState()
}

create()
newGame()
</script>

<template>
  <div>
    <h2 text-3xl>
      mime
    </h2>
    <div w-full mx-auto>
      <button class="bg-sky-500/75" px-20px py-10px color-white rounded @click="newGame()">
        New Game
      </button>
    </div>
    <div w-250px text-center mx-auto py-3>
      <div flex justify-between text-2xl>
        <div class="timer" flex gap-2>
          <span>⏱</span>
          <span>{{ timer }}s</span>
        </div>
        <div class="zhadan" flex gap-2>
          <span>💣</span>
          <span>{{ MimeNumber }}</span>
        </div>
      </div>
    </div>
  </div>
  <div w-full select-none>
    <ul v-for="row, r in data" :key="r" flex align-mid justify-center>
      <template v-for="(item, j) in row" :key="j">
        <li
          v-if="!item || !item.flip" m-1px w-10 h-10 leading-10 bg-gray-100 border="1px solid #e7e7e8" color-black
          class="hover:bg-gray-500/20 dark:bg-gray-300/30 hover:dark:bg-gray-200/10 dark:color-white" cursor="pointer"
          @click="toggle(r, j)"
        />
        <li
          v-if="item?.flip" m-1px w-10 h-10 leading-10 bg-white-100 border="1px solid #e7e7e8" color-black
          class="hover:bg-gray-100/10 dark:bg-gray-300/30 hover:dark:bg-gray-200/10 dark:color-white"
        >
          <template v-if="item?.value === '💣'">
            <span>{{ item.value }}</span>
          </template>
          <template v-else>
            <span :class="item.color">{{ item!.value || '' }}</span>
          </template>
        </li>
      </template>
    </ul>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
