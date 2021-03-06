<template>
  <ScrollView orientation="vertical">
    <StackLayout>
      <SegmentedBar class="segmented-bar m-x-5 m-t-10 m-b-5 p-x-10" v-model="selectedBarIndex" @selectedIndexChange="confirmSpymaster">
        <SegmentedBarItem title="Agent"/>
        <SegmentedBarItem title="Spymaster"/>
      </SegmentedBar>
      <FlexboxLayout class="game-grid">
        <Label
          v-for="(team,word) in words"
          :key="word"
          :text="word"
          @tap="flipCard(word)"
          class="game-tile text-center m-5 p-x-15 p-y-20"
          :class="getClass(word)"
        />
      </FlexboxLayout>
    </StackLayout>
  </ScrollView>
</template>

<script>
  import { mapState } from 'vuex';

  export default {
    data () {
      return {
        selectedBarIndex: 0,
        spymasterConfirmed: false,
      }
    },
    computed: {
      ...mapState(['gameboard', 'room']),
      isSpymaster () {
        return this.selectedBarIndex && this.spymasterConfirmed
      },
      words () {
        return this.gameboard.game.board
      },
      solution () {
        return this.gameboard.game.solution
      }
    },
    methods: {
      getClass (word) {
        let className = ''
        if (this.isSpymaster || this.words[word]) { // TODO check for spymaster prop
          className = this.solution[word].toLowerCase() + '-tile'
          if (this.isSpymaster && this.words[word]) { className = 'inverse-' + className }
        }
        return className
      },
      confirmSpymaster (o) {
        if (this.selectedBarIndex == 1) {
          confirm("Become spymaster?")
          .then(result => {
            if (result) {
              this.spymasterConfirmed = true
            } else {
              this.selectedBarIndex = 0
            }
          })
        }
      },
      flipCard (card) {
        // check if player is a spymaster
        if (this.isSpymaster) {
          // confirm that they want to flip
          confirm(`Flip "${card.trim()}" card?`)
          .then(result => {
            // flip the card
            if (result) {
              const params = {
                card: card,
                room: this.room,
              };
              this.$socketio.emit('flip_card', params);
            }
          })
        } else {
          alert('Only spymasters can flip cards.')
        }
      }
    }
  }
</script>

<style scoped>
  .game-grid {
    align-items: flex-start;
    flex-wrap: wrap;
  }
  .game-tile {
    flex-shrink: 0;
    flex-grow: 1;
    background-color: #424242;
    color: #eeeeee;
    font-size: 22px;
  }

  .game-tile.b-tile {
    background-color: #1976d2;
    color: white;
  }
  .game-tile.r-tile {
    background-color: #c62828;
    color: white;
  }
  .game-tile.o-tile {
    background-color: #ececec;
    color: #424242;
  }
  .game-tile.x-tile {
    background-color: #212121;
    color: white;
  }

  .game-tile.inverse-b-tile {
    color: #1976d2;
    background-color: #424242;
  }
  .game-tile.inverse-r-tile {
    color: #c62828;
    background-color: #424242;
  }
  .game-tile.inverse-o-tile {
    color: #ececec;
    background-color: #424242;
  }
  .game-tile.inverse-x-tile {
    color: #212121;
    background-color: #424242;
  }
</style>
