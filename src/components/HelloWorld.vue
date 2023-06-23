<script setup>
import { Subject, lastValueFrom, switchMap } from 'rxjs';
import { fromFetch } from 'rxjs/fetch';
import { ref } from 'vue';

defineProps({
  msg: String,
})


const onPromise = () => new Promise((resolve) => resolve(1))



const promise = async () => {
  onPromise().then(data => {
    console.log('promise then', data)
  })

  const data = await onPromise()
  console.log('OnPromise await', data)
}










const count = ref(0)
const subject$ = new Subject()

subject$.subscribe(data => {
  console.log('subject received ', data)
})

const increase = () => {
  subject$.next(count.value++)
}









// const obs$ = new Observable((subscribe) => {
//     subscribe.next(1)
//     subscribe.next(2)
//     subscribe.next(3)
//     subscribe.next(4)
//     subscribe.next(5)
// })
// obs$.subscribe(data => {
//   console.log('subscribe ', data)
// })










const data$ = fromFetch('https://api.github.com/users?per_page=5')
const getUser$ = (id = 1) => fromFetch(`https://jsonplaceholder.typicode.com/users/${id}`)

const fromFetchData = () => {
  data$.subscribe(async data => {
    if (data.ok) {
      const json = await data.json()
      console.log('json await', json)
    }
  })
}

const fromFetchSwitchMap = () => {
  data$.pipe(
    switchMap(resp => {
      if(resp.ok) return resp.json()
    })
  ).subscribe(data => {
    console.log('json switchMap', data)
  })
}

const fromFetchLastvalue = async () => {
  const onFetch = fromFetch('https://api.github.com/users?per_page=5')
  const resp = await lastValueFrom(onFetch)
  if(resp.ok) {
    const json = await resp.json()
    console.log('lastValueFrom', json)
  }
}

const witchMap2Req = () => {
  const currentId$ = fromFetch('https://jsonplaceholder.typicode.com/photos').pipe(
    switchMap(resp => resp.json())
  )
  currentId$.pipe(
    switchMap(({ id }) => {
      return getUser$(id).pipe(switchMap(resp => {
        if(resp.ok) return resp.json()
      }))
    })
  ).subscribe(data => {
    console.log('json witchMap2Req', data)
  })
}

</script>

<template>
  <h1>{{ msg }}</h1>
   <button @click="promise">Promise</button>
  <button @click="increase">Subject</button>
  <button @click="fromFetchData">fromFetch await</button>
  <button @click="fromFetchSwitchMap">SwitchMap</button>
  <button @click="fromFetchLastvalue">LastValueFrom</button>
  <button @click="witchMap2Req">SwitchMap2Req</button>
</template>

