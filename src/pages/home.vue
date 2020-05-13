<template>
  <div class="flex-column flex-flex-start-center p-16">
    <div class="bg-blue-45 p-25 m-t-20 border-radius-4">
      <h3>开关装饰器测试</h3>
      <p class="m-t-20"> 开关装饰器,控制一个变量,根据业务灵活控制,loading disabled 等 </p>
      <div class="flex-between-center">
        <van-button
          type="primary"
          class="m-t-10"
          :loading="loading"
          loading-text="loading 效果"
          @click="loadingTest"
        >
          loading 效果
        </van-button>
        <van-button
          class="m-l-10"
          type="primary"
          :disabled="loading"
          @click="loadingTest"
        >
          disabled 效果
        </van-button>
      </div>
      <p class="m-t-20">开关装饰器 自带loading 效果 全局效果 </p>
      <van-button class="m-t-10" type="info" @click="loadingTwo">全局加载装饰器默认提示</van-button>
      <p class="m-t-20">文案自定义</p>
      <van-button class="m-t-10" type="default" @click="loadingThree">全局加载提装饰器示自定义测试</van-button>
      <h3 class="m-t-50">confirm 装饰器测试</h3>
      <p class="m-t-20">普通confirm 只需传入confirm文案</p>
      <van-button class="m-t-10" type="danger" @click="testConfirm">confirm 装饰器测试</van-button>
      <p class="m-t-20">高级配置confirm 可传入整个option</p>
      <van-button class="m-t-10" type="warning" @click="testConfirmTwo">confirm 装饰器高级配置测试</van-button>
      <h3 class="m-t-50">字典测试</h3>
      <p class="m-t-20">通过name加载对象</p>
      <p>{{ dict }}</p>
      <p class="m-t-20">通过name加载数组 多用于下拉选择</p>
      <p>{{ dict2 }}</p>
      <p class="m-t-20">通过name，key加载具体项</p>
      <p>{{ dictItem }}</p>
    </div>
    <demo :test="{}" />
  </div>
</template>
<script>
  import { loading, confirm, toastLoading } from '@/decorator'
  import { getDictObj, getDictList, getDictItem } from '@/dicts'

  export default {
    name: 'Home',
    data () {
      return {
        loading: false,
        dict: getDictObj('taskType'),
        dict2: getDictList('taskType'),
        dictItem: getDictItem('taskType', 1)
      }
    },
    methods: {
      @loading('loading')
      loadingTest () {
        return new Promise((resolve, reject) => {
          setTimeout(resolve, 3000)
        })
      },

      @toastLoading()
      loadingTwo () {
        return new Promise((resolve, reject) => {
          setTimeout(resolve, 1500)
        })
      },
      @toastLoading('我是加载文字')
      loadingThree () {
        return new Promise((resolve, reject) => {
          setTimeout(resolve, 2000)
        })
      },
      @confirm('妥妥的')
      testConfirm () {
        console.log('确认')
      },
      @confirm({ title: '这是title', message: '内容主题' })
      testConfirmTwo () {
        console.log('确认')
      }
    }
  }
</script>
<style scoped lang="scss">
</style>
