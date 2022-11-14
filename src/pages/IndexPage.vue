<template>
  <q-page>
    <q-input filled v-model="text" placeholder="Deine Notiz">

      <template v-slot:after>
        <q-btn round dense flat icon="send" @click="saveNotice"/>
      </template>
    </q-input>

    <div class="row">
      <q-chip>
        <q-avatar color="warning" text-color="white">{{storedNotes.filter(notice => notice.done === false).length}}</q-avatar>
        Offen
      </q-chip>
      <q-chip>
        <q-avatar color="primary" text-color="white">{{storedNotes.filter(notice => notice.done === true).length}}</q-avatar>
        Erledigt
      </q-chip>
    </div>

    <q-list
      separator
      v-if="storedNotes.length > 0"
    >
      <q-item
        clickable
        v-for="(notice, i) in storedNotes"
        :key="i"
        @right="onRight(notice)"
        @click="done(notice)"
        :class="{'done bg-grey-4': notice.done }">

        <q-item-section avatar>
          <q-checkbox  v-model="notice.done"/>
        </q-item-section>
        <q-item-section>{{ notice.noticeText }}</q-item-section>

        <q-btn
          v-if="notice.done"
          @click.stop="onRight(notice)"
          flat
          round
          dense
          color="primary"
          icon="delete"
        />
      </q-item>
    </q-list>

    <div v-else class="pb-empty-list">
      <p class="text-center">Deine Liste ist noch leer.</p>
    </div>
  </q-page>
</template>

<script>
import {defineComponent, ref} from 'vue'

export default defineComponent({
  name: 'IndexPage',
  setup () {
    return {
      text: ref(''),
      noticeArray: ref([]),
      storedNotes: ref([]),
    }
  },
  methods: {
    saveNotice() {
      let newNotice = {
        id: `${Date.now()}-${Math.floor(Math.random() * 1000)}`,
        date: Date.now(),
        noticeText: this.text,
        done: false
      }
      let theNoticeArray = JSON.parse(localStorage.getItem('noticeArray'))

      if(theNoticeArray === null){
        this.noticeArray.push(newNotice)
        this.text = ''
        const jsonArr = JSON.stringify(this.noticeArray)
        localStorage.setItem('noticeArray', jsonArr)
        this.getNotes()
      } else {
        this.noticeArray = [...theNoticeArray]
        this.noticeArray.unshift(newNotice)
        const jsonArr = JSON.stringify(this.noticeArray)
        localStorage.setItem('noticeArray', jsonArr)
        this.text = ''
        this.getNotes()
      }
    },

    getNotes(){
      this.$q.loading.show()
      const theStoredNotes = localStorage.getItem('noticeArray')
      this.storedNotes = JSON.parse(theStoredNotes)
      this.$q.loading.hide()
    },

    done(notice){
      notice = {
        ...notice,
        done: !notice.done
      }
      let noticeIndex = this.storedNotes.findIndex((note) => note.id === notice.id);
      this.storedNotes[noticeIndex] = notice
      const jsonArr = JSON.stringify(this.storedNotes)
      localStorage.setItem('noticeArray', jsonArr)
    },

    onRight (notice) {
      let noticeIndex = this.storedNotes.findIndex((note) => note.id === notice.id);
      this.storedNotes.splice(noticeIndex, 1)
      const jsonArr = JSON.stringify(this.storedNotes)
      localStorage.setItem('noticeArray', jsonArr)
      this.getNotes()
    },
  },
  created() {
   this.getNotes()
  },
})
</script>

<style lang="scss">
.pb-empty-list{
  height: 70vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.done{
  .q-item__section--main{
    text-decoration: line-through;
    color: $dark;
  }
}
</style>
