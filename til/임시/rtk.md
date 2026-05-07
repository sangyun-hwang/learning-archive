# RTK



### createAction

```js
//기존
const ADD_TODO = 'todos/add';

const addTodo = (payload) => ({
    type: ADD_TODO,
    payload
})

// rtk
const addTodo = createAction('todos/add');

store.dispatch(addTodo({
    doit: 'RTK 다뤄보기'
}))

addTodo.type // 'todos/add'
addTodo.toString() // 'todos/add'


// createAction 내부 구조
function createAction(type, prepareFn) {
  const action = { type };
  const actionCreator = (payload) => {
    if (payload) action.payload = payload;
    const preapreAction = prepareFn?.(payload) ?? {};
    return { ...action, ...preapreAction };
  };
  actionCreator.type = type;
  actionCreator.toString = () => type;
  return actionCreator;
}

export default createAction;

```

```js
const { uniqueId } = require("lodash");

const addTodo = createAction('todos/add', ({doit}) => {
    return {
        id: uniqueId(),
        doit,
        completed: false,
    }
});
```



### slices

기능마다 분리

![image-20221024060139179](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20221024060139179.png)

- 기본 구성

![image-20221024060317384](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20221024060317384.png)





### createSlice

```js
import { createSlice, createSelector } from '@reduxjs/toolkit';

const slice = createSlice({
  name: 'todoList', // 리덕스에서 관리하는 상태 이름
  initialState: [], // 초기 상태
  reducers: { // 여러개의 리듀서 병합
    addTodo(state, action) {
      state.push({
        ...action.payload,
        completed: false,
      });
    },
    toggleTodo(state, action) {
          return state.map((todo) => {
                return {
                  ...todo,
                  completed: !todo.completed,
                };
      });
    },
  },
});

console.log(slice); // { actions, reducer }

export const { addTodo, toggleTodo } = slice.actions;

// todoListReducer
export default slice.reducer;
```





### index.js

```js
import { configureStore } from '@reduxjs/toolkit';

// root reducer: 리듀서 병합
const reducers = {
  todoList,
  visibilityFilter,
  beverageList,
};


// store options
const preloadedState = { // 초기 값
  todoList: [
    { id: 'learn-react', doit: 'React 학습', completed: true },
    { id: 'learn-redux', doit: 'Redux 학습', completed: true },
    {
      id: 'learn-rtk',
      doit: 'Redux Toolkit 학습',
      completed: false,
    },
  ],
};


// store
const store = configureStore({
  reducer: reducers, // 병합된 리듀서 호출
  preloadedState: loadState() ?? preloadedState,
  devTools: true,
});

export default store;
```





### createAsyncThunk() - 비동기 호출용

- pending:  대기
- fulfilled: 요청 성공
- rejected: 거절 (오류 핸들링)



```js
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchBeverageList = createAsyncThunk(
  'beverage-list/fetch',
  async () => {
    const { data } = await axios('/beverage-menu');
    return data;
  }
);

export const fetchBeverageItem = createAsyncThunk(
  'beverage-item/fetch',
  async (id) => {
    const { data } = await axios(`/beverage-menu/${id}`);
    return data;
  }
);

export const fetchSearchBeverage = createAsyncThunk(
  'search-baverage/fetch',
  async (query) => {
    const { data } = await axios.get(`/beverage-menu?q=${query}`);
    return data;
  }
);
```



- createSlice
  - reducers: sync
  - extraReducers: async(비동기)

```js
const slice = createSlice({
  name: 'beverageList',
  initialState: {
    loading: true,
    error: null,
    list: null,
    item: null,
  },
  extraReducers: (builder) => {
    const pendingCallback = (state) => { // 대기
      state.loading = true; // 로딩 상태 관리
    };

    const rejectedCallback = (state, action) => { // 거절
      state.loading = false; // 로딩 상태 관리
      state.error = action.payload; // 에러 처리
    };

    builder
      .addCase(fetchBeverageList.pending, pendingCallback)
      .addCase(fetchBeverageList.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })
      .addCase(fetchBeverageList.rejected, rejectedCallback)
      // 응답 상태에 따라 케이스 구분
      .addCase(fetchBeverageItem.pending, pendingCallback)
      .addCase(fetchBeverageItem.fulfilled, (state, action) => {
        state.loading = false;
        state.item = action.payload;
      })
      .addCase(fetchBeverageItem.rejected, rejectedCallback)
      .addCase(fetchSearchBeverage.pending, pendingCallback)
      .addCase(fetchSearchBeverage.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })
      .addCase(fetchSearchBeverage.rejected, rejectedCallback);
  },
});

export default slice.reducer;
```

![image-20221024064742321](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20221024064742321.png)



- useDispatch: 상태값 업데이트
- useSelector: 상태값 불러오기





- 커스텀 훅

```js
export const useBeverageList = () => {
  const { loading, error, list } = useSelector(
    selectBeverageList
  );

  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchBeverageList()/* async thunk function */);
  }, [dispatch]);

  return { loading, error, list };
};

export const useBeverageItem = (id) => {
  const { loading, error, item } = useSelector(
    ({ beverageList }) => beverageList
  );

  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchBeverageItem(id));
  }, [id, dispatch]);

  return { loading, error, item };
};


// 성능 관리(캐시: 메모이제이션 활용)
// 캐시된 상태 값이 변경 된다면 다시 가져오기
// 캐시된 상태 값이 변경되지 않았다면 캐시된 데이터 가져오기
export const selectBeverageList = createSelector(
  (state) => state.beverageList,
  (beverageList) => beverageList
);
```







# RTK query

![image-20221024063149746](https://raw.githubusercontent.com/ddullgi/image_sever/master/img/image-20221024063149746.png)

- createSlice & createAsyncThunk를 사용해서 만들어





### createApi, fetchBaseQuery

- 우리가 훅 만들필요 없이 만들어줌

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const contactsApi = createApi({
  reducerPath: 'contactsApi',
  baseQuery: fetchBaseQuery({
    baseUrl: 'http://localhost:4000',
  }), // 기본 주소
  tagTypes: ['Contact'],
  endpoints: (builder) => ({
    // QUERY 가져오기
    contacts: builder.query({
      query: () => '/contacts',
        // 다시 가져옴
      providesTags: ['Contact'],
    }),
    contact: builder.query({
      query: (id) => {
        if (!id) return '';
        return `/contacts/${id}`;
      },
      providesTags: ['Contact'],
    }),

    // MUTATION 쓰기
    addContact: builder.mutation({
      query: (newContact) => ({
        url: '/contacts',
        method: 'POST',
        body: newContact,
      }),
      // 다시 가져오지 않음
      invalidatesTags: ['Contact'],
    }),
    updateContact: builder.mutation({
      query: ({ id, ...willUpdateContact }) => ({
        url: `/contacts/${id}`,
        method: 'PUT',
        body: willUpdateContact,
      }),
      invalidatesTags: ['Contact'],
    }),
    deleteContact: builder.mutation({
      query: (id) => ({
        url: `/contacts/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['Contact'],
    }),
  }),
});

export const {
    // 가져오기
  useContactsQuery,
  useContactQuery,
    // 쓰기
  useAddContactMutation,
  useUpdateContactMutation,
  useDeleteContactMutation,
} = contactsApi;
```
