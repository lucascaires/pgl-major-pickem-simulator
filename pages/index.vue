<template>
  <div class="min-h-screen flex flex-col justify-center bg-gray-200">
    <h1 class="text-indigo-500 text-center text-xl lg:text-4xl py-10">PGL Major Stockholm - Pick'em Simulator</h1>
    <div class="w-full max-w-5xl mx-auto p-4 lg:p-0">
      <div class="grid lg:grid-cols-2 gap-4">
        <div class="bg-white shadow p-4 rounded-lg" v-if="!duels.length">
          <h2 class="text-xl text-indigo-500 mb-4">Finished</h2>
        </div>
        <div class="bg-white shadow p-4 rounded-lg" v-if="duels.length">
          <h2 class="text-xl text-indigo-500 mb-4">Round {{ round }}</h2>
          <p class="text-sm text-gray-600 mb-4">Just click on a team name to pick a winner</p>
          <div class="divide-y">
            <div class="flex justify-between items-center" v-for="([teamA, teamB], key) in duels" :key="key">
              <div class="p-2 flex-1 text-center">
                <button
                  class="p-2 text-sm rounded"
                  @click="pick(key, teamA)"
                  :class="{
                    'opacity-50 pointer-events-none': selectedDuels.includes(key),
                    'bg-green-500 text-white': roundWinner.includes(teamA),
                    'bg-red-500 text-white': roundLoser.includes(teamA),
                  }"
                >
                  {{ teamA.name }}
                </button>
              </div>
              <div class="flex-1 text-center text-xl text-indigo-500">
                x
              </div>
              <div class="p-2 flex-1 text-center">
                <button
                  class="p-2 text-sm rounded"
                  @click="pick(key, teamB)"
                  :class="{
                    'opacity-50 pointer-events-none': selectedDuels.includes(key),
                    'bg-green-500 text-white': roundWinner.includes(teamB),
                    'bg-red-500 text-white': roundLoser.includes(teamB),
                  }"
                >
                  {{ teamB.name }}
                </button>
              </div>
            </div>
          </div>
          <button
            class="mt-4 bg-indigo-500 text-white rounded-lg p-4 w-full text-lg"
            @click.prevent="next"
            :class="{
              'opacity-50 pointer-events-none': selectedDuels.length < duels.length,
            }"
          >
            Next round
          </button>
        </div>
        <div class="bg-white shadow p-4 rounded-lg">
          <h2 class="text-xl text-indigo-500 mb-4">Results</h2>
          <div class="space-y-2">
            <div v-for="each in ['3-0', '3-1', '3-2', '0-3', '1-3', '2-3']" :key="each">
              <h3
                class="text-white rounded p-2"
                :class="{
                  'bg-green-500': ['3-0', '3-2', '3-1'].includes(each),
                  'bg-red-500': ['0-3', '1-3', '2-3'].includes(each),
                }"
              >
                {{ each }}
              </h3>
              <p v-if="!state[each] || !state[each].length" class="p-1 text-sm text-gray-700">No teams yet</p>
              <div v-if="state[each] && state[each].length">
                <ul class="divide-y">
                  <li v-for="{ name } in state[each]" :key="name" class="p-1 text-sm text-gray-700">{{ name }}</li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const INITIAL_SEEDS = ['Team Spirit', 'Astralis', 'paiN Gaming', 'ENCE', 'BIG', 'Movistar Riders', 'Heroic', 'MOUZ', 'Sharks Esports', 'TYLOO', 'Renegades', 'Entropiq', 'GODSENT', 'Virtus.pro', 'Copenhagen Flames', 'FaZe Clan']
export default {
  data() {
    return {
      round: 1,
      teams: [],
      state: [],
      duels: [],
      selectedDuels: [],
      roundWinner: [],
      roundLoser: [],
    }
  },
  computed: {},
  methods: {
    setState() {
      const result = {}
      ;['0-1', '0-2', '0-3', '1-0', '1-1', '1-2', '2-0', '2-1', '2-2', '2-3', '3-0', '3-1', '3-2'].forEach((each) => {
        const [wins, losses] = each.split('-').map((e) => parseInt(e))
        result[each] = this.teams.filter((e) => e.wins == wins && e.losses == losses)
      })
      this.state = { ...result }
    },

    pick(index, team) {
      const winner = team
      const loser = this.duels[index].filter((e) => e !== team)[0]
      this.roundWinner.push(winner)
      this.roundLoser.push(loser)
      this.teams = this.teams.map((e) => {
        const addWins = e.name === winner.name ? 1 : 0
        const addLosses = e.name === loser.name ? 1 : 0
        return {
          name: e.name,
          wins: e.wins + addWins,
          losses: e.losses + addLosses,
        }
      })
      this.selectedDuels.push(index)
    },

    mountInitialTeams() {
      this.teams = INITIAL_SEEDS.map((team) => {
        return {
          name: team,
          wins: 0,
          losses: 0,
        }
      })
    },

    mountInitialDuels() {
      for (let i = 0; i <= (INITIAL_SEEDS.length - 1) / 2; i++) {
        this.duels.push([this.teams[i], this.teams[INITIAL_SEEDS.length - 1 - i]])
      }
    },

    mountNextDuels() {
      this.duels = []
      Object.keys(this.state)
        .filter((each) => this.state[each].length)
        .forEach((key) => {
          if (!['0-3', '1-3', '2-3', '3-0', '3-1', '3-2'].includes(key)) {
            const each = this.state[key]
            const seededTeams = each.sort((a, b) => {
              const aIndex = INITIAL_SEEDS.findIndex((each) => each === a.name)
              const bIndex = INITIAL_SEEDS.findIndex((each) => each === b.name)
              return aIndex - bIndex
            })
            this.duels = [...this.duels, ...this.mountDuels(seededTeams)]
          }
        })
    },

    mountDuels(collection) {
      const duels = []
      for (let i = 0; i <= (collection.length - 1) / 2; i++) {
        duels.push([collection[i], collection[collection.length - 1 - i]])
      }
      return duels
    },

    next() {
      this.round = this.round + 1
      this.setState()
      this.mountNextDuels()
      this.selectedDuels = []
      this.roundWinner = []
      this.roundLoser = []
    },
  },
  created() {
    this.mountInitialTeams()
    this.mountInitialDuels()
  },
}
</script>
