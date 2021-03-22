#### Composition API를 사용하는 이유
- 관련있는 로직들이 매우 떨어져 있기 때문에 처음보는 사람들이 이해하기가 쉽지 않다

#### Composition API 사용방법
- `setup`이라는 컴포넌트 옵션을 사용한다
- `setup`은 컴포넌트가 생성 되기전에 실행된다
- `setup`은 this를 가지고 있지 않다
- `setup`은 local state, computed properties, methods에 접근 불가능하다
- `setup`은 props와 context를 인자로 받는다
- `setup` 기본 구조
  ```js
    export default {
      components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
      props: {
        user: {
          type: String,
          required: true
        }
      },
      setup(props) {
        console.log(props) // { user: '' }

        return {} // anything returned here will be available for the rest of the component
      }
      // the "rest" of the component
    }
  ```
  ```js
  // src/components/UserRepositories.vue `setup` function
  import { fetchUserRepositories } from '@/api/repositories'

  // inside our component
  setup (props) {
    let repositories = []
    const getUserRepositories = async () => {
      repositories = await fetchUserRepositories(props.user)
    }

    return {
      repositories,
      getUserRepositories // functions returned behave the same as methods
    }
  }
  ```
- `setup`안에서 Reactive특성을 가진 local data를 만들기 위해선 `ref`를 사용한다
  ```js
  import { ref } from '@vue/composition-api'

  const counter = ref(0)
  ```
- `ref`에서 강제로 object를 만들고 value를 넣어주는 이유는 JavaScript에서 number | string과 같은 참조가 아닌 값으로 전달하기 위해서이다
  ```js
  import { ref } from '@vue/composition-api'

  const counter = ref(0)

  console.log(counter) // { value: 0 }, 자동으로 value라는 키값을 가지게 된다.
  console.log(counter.value) // 0

  counter.value++
  console.log(counter.value) // 1
  ```
  ```js
  import { fetchUserRepositories } from '@/api/repositories'
  import { ref } from '@vue/composition-api'

  // in our component
  setup (props) {
    const repositories = ref([]) // ref 사용
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(props.user) // ref.value에 값넣어줌
    }

    return {
      repositories, // 반환
      getUserRepositories
    }
  }
  ```
- `setup`에서 Lifecycle Hook 사용법
  ```js
  import { fetchUserRepositories } from '@/api/repositories'
  import { ref, onMounted } from 'vue'

  // in our component
  setup (props) {
    const repositories = ref([])
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(props.user)
    }

    onMounted(getUserRepositories) // on `mounted` call `getUserRepositories`

    return {
      repositories,
      getUserRepositories
    }
  }
  ```
- `setup`에서 watch 사용법
  ```js
  import { ref, watch } from '@vue/composition-api'

  const counter = ref(0)
  watch(counter, (newValue, oldValue) => {
    console.log('The new counter value is: ' + counter.value)
  })
  ```
- `setup`에서 computed 사용법
  ```js
  import { ref, computed } from '@vue/composition-api'

  const counter = ref(0)
  const twiceTheCounter = computed(() => counter.value * 2)

  counter.value++
  console.log(counter.value) // 1
  console.log(twiceTheCounter.value) // 2
  ```
- `setup`안에 option들 코드를 다 집어넣음으로서 매우 커지기 때문에 분리해줘야 된다
  ```js
  // 분리하기 전 코드
  // src/components/UserRepositories.vue `setup` function
  import { fetchUserRepositories } from '@/api/repositories'
  import { ref, onMounted, watch, toRefs, computed } from 'vue'

  // in our component
  setup (props) {
    // using `toRefs` to create a Reactive Reference to the `user` property of props
    const { user } = toRefs(props)

    const repositories = ref([])
    const getUserRepositories = async () => {
      // update `props.user` to `user.value` to access the Reference value
      repositories.value = await fetchUserRepositories(user.value)
    }

    onMounted(getUserRepositories)

    // set a watcher on the Reactive Reference to user prop
    watch(user, getUserRepositories)

    const searchQuery = ref('')
    const repositoriesMatchingSearchQuery = computed(() => {
      return repositories.value.filter(
        repository => repository.name.includes(searchQuery.value)
      )
    })

    return {
      repositories,
      getUserRepositories,
      searchQuery,
      repositoriesMatchingSearchQuery
    }
  }
  ```
  ```js
  // src/composables/useUserRepositories.js <- 이 파일로 따로 빼줌
  import { fetchUserRepositories } from '@/api/repositories'
  import { ref, onMounted, watch } from 'vue'

  export default function useUserRepositories(user) {
    const repositories = ref([])
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(user.value)
    }

    onMounted(getUserRepositories)
    watch(user, getUserRepositories)

    return {
      repositories,
      getUserRepositories
    }
  }
  ```
  ```js
  // src/composables/useRepositoryNameSearch.js <- 이 파일로 따로 빼줌
  import { ref, computed } from 'vue'

  export default function useRepositoryNameSearch(repositories) {
    const searchQuery = ref('')
    const repositoriesMatchingSearchQuery = computed(() => {
      return repositories.value.filter(repository => {
        return repository.name.includes(searchQuery.value)
      })
    })

    return {
      searchQuery,
      repositoriesMatchingSearchQuery
    }
  }
  ```
  ```js
  // 최종 코드
  // src/components/UserRepositories.vue
  import { toRefs } from 'vue'
  import useUserRepositories from '@/composables/useUserRepositories'
  import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
  import useRepositoryFilters from '@/composables/useRepositoryFilters'

  export default {
    components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
    props: {
      user: {
        type: String,
        required: true
      }
    },
    setup(props) {
      const { user } = toRefs(props)

      const { repositories, getUserRepositories } = useUserRepositories(user)

      const {
        searchQuery,
        repositoriesMatchingSearchQuery
      } = useRepositoryNameSearch(repositories)

      // 이 부분은 정리에 나와있지 않다.
      const {
        filters,
        updateFilters,
        filteredRepositories
      } = useRepositoryFilters(repositoriesMatchingSearchQuery)

      return {
        // Since we don’t really care about the unfiltered repositories
        // we can expose the end results under the `repositories` name
        // repositories 같은 경우는 어차피 computed 값만 쓰므로 이렇게 넣어주도록 한다.
        repositories: filteredRepositories,
        getUserRepositories,
        searchQuery,
        filters,
        updateFilters
      }
    }
  }
  ```



