<script setup lang="ts">
import { computed, ref, watch, Ref } from 'vue'
import Test from './Test.vue'
defineProps<{ msg: string }>()
type Person = {
  name: string
  age: number
  nested: {
    foo: string
    fee: {
      a: 1
    }
  }
  optional?: {
    bar: string
  }
  optional2?: string
}
const data: Ref<Person> = ref({
  name: 'John Doe',
  age: 30,
  nested: {
    foo: 'abc',
    fee: {
      a: 1
    }
  }
})
// // Returns R if T is a function, otherwise returns Fallback
// type IsFunction<T, R, Fallback = T> = T extends (...args: any[]) => any
//   ? R
//   : Fallback

// // Returns R if T is an object, otherwise returns Fallback
// type IsObject<T, R, Fallback = T> = IsFunction<
//   T,
//   Fallback,
//   T extends object ? R : Fallback
// >

// // "a.b.c" => "b.c"
// type Tail<S> = S extends `${string}.${infer T}` ? Tail<T> : S

// // typeof Object.values(T)
// type Value<T> = T[keyof T]
// // {a: {b: 1, c: 2}} => {"a.b": {b: 1, c: 2}, "a.c": {b: 1, c: 2}}
// type FlattenStepOne<T> = {
//   [K in keyof T as K extends string
//     ? IsObject<T[K], `${K}.${keyof T[K] & string}`, K>
//     : K]: IsObject<T[K], { [key in keyof T[K]]: T[K][key] }>
// }

// // {"a.b": {b: 1, c: 2}, "a.c": {b: 1, c: 2}} => {"a.b": {b: 1}, "a.c": {c: 2}}
// type FlattenStepTwo<T> = {
//   [a in keyof T]: IsObject<
//     T[a],
//     Value<{ [M in keyof T[a] as M extends Tail<a> ? M : never]: T[a][M] }>
//   >
// }

// // {a: {b: 1, c: {d: 1}}} => {"a.b": 1, "a.c": {d: 1}}
// type FlattenOneLevel<T> = FlattenStepTwo<FlattenStepOne<T>>

// // {a: {b: 1, c: {d: 1}}} => {"a.b": 1, "a.b.c.d": 1}
// type Flatten<T> = T extends FlattenOneLevel<T>
//   ? T
//   : Flatten<FlattenOneLevel<T>> & { [K in keyof T]: T[K] }

// type NestedKeyOf<T, K = keyof T> = K extends keyof T & (string | number)
//   ? `${K}` | (T[K] extends object ? `${K}.${NestedKeyOf<T[K]>}` : never)
//   : never
type Writable<T, O> = T extends O
  ? T
  : {
      [P in keyof T as IfEquals<
        { [Q in P]: T[P] },
        { -readonly [Q in P]: T[P] },
        P
      >]: T[P]
    }

type IfEquals<X, Y, A = X, B = never> = (<T>() => T extends X ? 1 : 2) extends <
  T
>() => T extends Y ? 1 : 2
  ? A
  : B
type Cleanup<T> = 0 extends 1 & T
  ? unknown
  : T extends readonly any[]
  ? Exclude<keyof T, keyof any[]> extends never
    ? { [k: `${number}`]: T[number] }
    : Omit<T, keyof any[]>
  : T

type PrefixKeys<V, K extends PropertyKey, O> = V extends O
  ? { [P in K]: V }
  : V extends object
  ? {
      [P in keyof V as `${Extract<K, string | number>}.${Extract<
        P,
        string | number
      >}`]: V[P]
    }
  : { [P in K]: V }

type ValueOf<T> = T[keyof T]
type Flatten2<T, O = never> = Writable<Cleanup<T>, O> extends infer U
  ? U extends O
    ? U
    : U extends object
    ?
        | ValueOf<{
            [K in keyof U]-?: (x: PrefixKeys<Flatten2<U[K], O>, K, O>) => void
          }>
        | ((x: U) => void) extends (x: infer I) => void
      ? { [K in keyof I]: I[K] }
      : never
    : U
  : never

function genDynRef<T>(data: Ref<T>) {
  return <K extends keyof Flatten2<T> & string>(path: K) => {
    return computed({
      get() {
        const pathArr = path.split('.')
        let result = data.value as any
        for (const key of pathArr) {
          result = result[key]
        }
        return result as Flatten2<T>[typeof path]
      },
      set(value) {
        const pathArr = path.split('.')
        let result = data.value as any
        for (const key of pathArr.slice(0, -1)) {
          result = result[key]
        }
        result[pathArr[pathArr.length - 1]] = value
      }
    })
  }
}

watch(
  data,
  () => {
    console.log(data.value)
  },
  { deep: true }
)
// 紐づけたいデータとパスを指定すれば、v-modelで双方向バインディングできるようにする汎関数
const a = genDynRef<Person>(data)('nested.fee.a')

// formFactory(source, bindinfPath, formLen, formType) -> formObject
</script>

<template>
  <Test v-model="a"></Test>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
