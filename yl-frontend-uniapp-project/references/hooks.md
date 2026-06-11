# uni-app 核心 Hooks

所有项目使用 Composition API hooks 实现逻辑复用。

## usePage — 分页列表

最核心的 hook，所有列表页使用。

```ts
// hooks/usePage.ts
import { ref, reactive } from 'vue'
import { onShow } from '@dcloudio/uni-app'
import { http } from '@/utils/http'

export function usePage<T>(url: string, defaultParams = {}) {
  const list = ref<T[]>([])
  const loading = ref(false)
  const finished = ref(false)
  const pageNo = ref(1)
  const pageSize = ref(20)
  const total = ref(0)
  const params = reactive({ ...defaultParams })

  const loadData = async (reset = false) => {
    if (reset) {
      pageNo.value = 1
      finished.value = false
    }
    if (finished.value) return

    loading.value = true
    try {
      const res = await http({
        url,
        data: {
          current: pageNo.value,
          pageSize: pageSize.value,
          querys: buildQuerys(params),
        },
      })
      if (reset) {
        list.value = res.data.records
      } else {
        list.value.push(...res.data.records)
      }
      total.value = res.data.total
      if (list.value.length >= total.value) {
        finished.value = true
      }
      pageNo.value++
    } finally {
      loading.value = false
    }
  }

  const onRefresh = () => loadData(true)
  const onLoadMore = () => loadData()

  onShow(() => loadData(true))

  return {
    list, loading, finished, total, params,
    loadData, onRefresh, onLoadMore,
  }
}
```

### 页面中使用

```vue
<script setup lang="ts">
import { usePage } from '@/hooks/usePage'

const { list, loading, finished, onRefresh, onLoadMore } = usePage('/api/list')
</script>
```

---

## useEnum — 动态枚举

从后端获取枚举选项，带缓存：

```ts
// hooks/useEnum.ts
import { ref } from 'vue'
import { http } from '@/utils/http'

const cache = new Map<string, any[]>()

export function useEnum() {
  const getEnum = async (groupNo: number) => {
    if (cache.has(String(groupNo))) {
      return cache.get(String(groupNo))
    }
    const res = await http({ url: '/system/enum/get', data: { groupNo } })
    const options = res.data.map((item: any) => ({
      label: item.name,
      value: item.code,
    }))
    cache.set(String(groupNo), options)
    return options
  }

  return { getEnum }
}
```

---

## useDoc — 档案类型数据

获取部门、人员、角色等档案类型下拉数据：

```ts
// hooks/useDoc.ts
export function useDoc() {
  const getDeptList = () => http({ url: '/system/dept/query' })
  const getPersonalList = () => http({ url: '/system/personal/query' })
  const getRoleList = () => http({ url: '/system/role/query' })
  const getFeeItemList = () => http({ url: '/system/feeItem/query' })

  return { getDeptList, getPersonalList, getRoleList, getFeeItemList }
}
```

---

## useCurrentFP — 当前财年（部分项目）

```ts
export function useCurrentFP() {
  const currentFP = ref('')
  const initFP = async () => {
    const res = await http({ url: '/system/financePeriod/current' })
    currentFP.value = res.data
  }
  return { currentFP, initFP }
}
```

---

## useDeptTeamDefault — 部门班组默认值（部分项目）

```ts
export function useDeptTeamDefault() {
  // 从 Pinia store 获取当前用户的默认部门、班组
  const appStore = useAppStore()
  const defaultDept = computed(() => appStore.deptInfo?.deptId)
  const defaultTeam = computed(() => appStore.deptInfo?.teamId)

  return { defaultDept, defaultTeam }
}
```
