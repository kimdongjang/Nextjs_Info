# Nextjs_Info
Next js 정리  
https://kyounghwan01.github.io/blog/React/next/getInitialProps/#getinitialprops-%E1%84%8B%E1%85%B5%E1%84%8C%E1%85%A5%E1%86%B7   
ssr https://velog.io/@mollang/2020.01.23-%EC%84%9C%EB%B2%84%EC%82%AC%EC%9D%B4%EB%93%9C%EB%A0%8C%EB%8D%94%EB%A7%81-SSR-k8k5q7n38h  
redux https://github.com/kirill-konshin/next-redux-wrapper  
redux2 https://velog.io/@ansrjsdn/Next.js%EC%97%90%EC%84%9C-Redux-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0  
redux+ts https://jktech.tistory.com/46  
redux+ts2 https://sheldhe93.tistory.com/44  
redux+toolkit https://cotak.tistory.com/164  
redux+codepen https://codesandbox.io/s/next-redux-wrapper-demo-7n2t5?file=/components/store.tsx:230-239  
redux+saga https://github.com/dev-palboksoft/nextjs-typescript-redux-saga  
menu https://basketdeveloper.tistory.com/72  
routing https://salgum1114.github.io/nextjs/2019-05-24-nextjs-static-website-4/   


# Next.js
1. ```./pages/``` 폴더 안에 다른 폴더를 만들면 해당 라우팅의 페이지들은 서버측에서 먼저 로드해줌
  
## pre-rendering
생성된 각 HTML은 최소한의 자바스크립트 코드와 함께 생성되고, 그 js코드는 페이지가 완전히 인터렉티브하도록 한다.(ReactDOM.hydrate) 이것을 hydration이라고 한다.
+ Static Generation(정적 생성) : 개발자가 ```next build```를 실행했을 때 HTML을 생성한다. 보통 데이터에 의존하지 않는 (Data Fetching하지 않는 정적인 페이지들) 페이지들은 모두 이해 해당하며, 페이지의 컨텐츠나 경로가 외부 데이터에 의존할 경우 ```getStaticProps```와 getStaticPaths```를 활용해 HTML을 정적 생성할 수 있다. 이 정적 생성은 사용자의 상호작용에 관계 없이 똑같은 정보를 제공해야 하는 프로모션 페이지에 사용할 수 있다. 반면, User Request에 따라 데이터가 달라져야 한다면 Server-side Rendering을 사용한다.
+ Server-Side Rendering(SSR) : 매 요청마다 데이터를 fetching해야 한다면 ```getServerSideProps```를 사용해 SSR를 구성한다.
  

## getInitialProps
#### next v9 이상에서는 getInitialProps 대신 getStaticProps, getStaticPaths, getServerSideProps을 사용  
Data Fectching이라는 로직은 CSR에서 사용되는 react의 ```componentDidMount``` 또는 ```useEffect```로 컴포넌트가 마운트를 하고 나서 사용한다.  
이 과정을 서버에서 미리 처리하도록 도와주는 것이 getInitialProps이고, 내부 로직은 서버에서 실행되며 Client에서만 가능한 로직은 피해야 한다.(```Window```, ```document``` 등)  
```js
function Page({ stars }) {
  return <div>Next stars: {stars}</div>
}

Page.getInitialProps = async (ctx) => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const json = await res.json()
  return { stars: json.stargazers_count }
}

export default Page
```  
getInitialProps는 각 페이지에 static method로 구현한다.  
page의 Props의 starts는 client로 보내지기 전에 server 측에서 stats값을 받은 후에 보내진다.  
Context(ctx)는 아래와 같은 기본 구성을 가지고 있다.  
+ ```pathname``` - (string) 현재 route 이름
+ ```query``` - (object) url의 query string section
+ ```asPath``` - (string) query를 포함한 실제 경로명(브라우저에 보이는 주소값)
+ ```req``` - HTTP request object (server only)
+ ```res``` - HTTP response object (server only)
+ ```err``` - 렌더링 중 오류 발생시 Error object if any error is encountered during the rendering


### getStaticProps
서버 측에서만 실행되는 함수로 getStaticProps는 클라이언트 측에서 실행되지 않는다.
빌드 시에 딱 한 번만 호출 되며, static file로 빌드 되는데, 이 함수는 **API와 같은 외부 데이터**```(SQL, ...)```를 받아 Static Generation 하기 위한 용도다.
  
```js
// getStaticProps()에 의해 빌드시에 게시물이 채워진다.
function Blog({ posts }) {
  return (
    <ul>
      {posts.map(post => (
        <li>{post.title}</li>
      ))}
    </ul>
  );
}

export async function getStaticProps() {
  // 외부 API Endpoint로 Call해서 Post로 정보를 가져온다.
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  // posts 데이터가 담긴 prop를 빌드 시간에 Blog 컴포넌트로 전달한다.
  return {
    props: {
      posts
    }
  };
}

export default Blog;
```
+ params: 동적 경로를 사용하는 페이지에 대한 경로 정보를 담는다. 예를 들어, 동적 페이지 파일명이 [id].js인 경우 context.params에 {id:...}가 반환된다.
+ props: 페이지 컴포넌트에서 Props로 전달할 객체  


### getStaticPaths
getStaticPaths는 동적 라우팅으로 페이지를 동적으로 생성할 때, 특정 페이지는 정적으로 생성하고 싶을 때 사용한다.  
동적 라우팅(라우팅 되는 경우의 수 따져서 하위로 넣음)을 사용하며, 빌드 타임 때, 정적으로 렌더링할 경로 설정  
이곳에 정의하지 않은 하위 경로는 접근해도 페이지가 안뜸  
```js 
export async function getStaticPath() {
  return {
    paths: [
      { params: {...} }
    ],
    fallback: true or false
  };
}
```
+ paths : 어떤 경로를 pre-rendering할 것인지 명시
+ fallback :
  + false : paths에 포함되지 않은 404를 리턴
  + true : getStaticPaths에서 pre-rendering하지 않고 getStaticProps에서 하겠다는 의미.


### getServerSideProps
매 요청마다 페이지를 pre-rendering한다.
```js
export async function getServerSideProps(context){
  return {
    props: {} ,
  }
}
```
파라미터  
+ parmas : 동적 라우팅 사용시 받는 params
+ req : HTTP incomingMessage
+ res : HTTP response
+ query : query string
리턴값  
+ props : 컴포넌트가 받을 props
+ notFound : true일때 404 리턴
+ redirect : 리다이렉트할 페이지 경로



## ServerSide Cycle
1. Next Server가 GET 요청을 받는다.
2. 요청에 맞는 Page를 찾는다.
3. ```_app.js```의 ```getInitialProps```가 있다면 실행한다.
4. Page Component의 ```getInitialProps```가 있다면 실행한다. pageProps들을 받아온다.
5. ```_document.js```의 ```getInitialProps```가 있다면 실행한다. pageProps들을 받아온다.
6. 모든 props들을 구성하고, ```_app.js``` > page Component 순서로 rendering.
7. 모든 Content를 구성하고 ```_document.js```를 실행하여 html 형태로 출력한다.

```js
function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
 
```
#### Next 서버로 요청이 들어왔을때, 요청이 들어온 페이지에 들어갈 데이터를 Fetch하고 Html을 구성하여 client로 보내준다.  
#### 기본 구성 : ```_app.js``` / ```_document.js```
+ _app.js
```js
function MyApp({ Component, pageProps }) {
  return (
    <Layout>
    	<Component {...pageProps} />
	  </Layout>
  );
}

export default MyApp;
```
props로 받은 Component는 클라이언트가 요청한 페이지다. GET/ 요청을 받았다면 Component에는 ```/pages/index.js``` 파일을 props로 받는다.  
pageProps는 페이지 getInitialProps를 통해 내려 받은 props들을 말합니다.  

```js
import Document, { Html, Head, Main, NextScript } from 'next/document'

class MyDocument extends Document {
  
  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```
+ _document.js  
static html을 구성하기 위해 _app.js에서 구성한 html body가 어떤 형태로 구조화되어 생성되는지 구성하는 곳이다.


## Next Redux + Saga + Api 적용
### package
```
npm i redux
npm i redux-thunk
npm i @redux-devtools/extension
npm i next-redux-wrapper
```

### Redux의 타입 생성
#### Product.ts
```js
export interface Product {
    message: string;
    status: string;
  }
```
#### ProductApi.ts 
```js
export interface ProductApi {
    loading: boolean;
    data: Product;
    error: AxiosError | null;
}
```
### Redux Reducer+Action 생성(createSlice 사용)
#### productReducer.ts
```js
const initialState: ProductApi = {
    loading: false,
    data: { message:"https://images.dog.ceo/breeds/newfoundland/n02111277_7377.jpg", status:"test"},
    error: null    
}

const productsSlice = createSlice({
    name: "products",
    initialState,
    reducers: {
        // 액션에 따른 reducer 로직을 작성한다.
        // createSlice가 자동으로 state의 타입을 추론한다.
        // 또한 immer를 사용하고 있어 함수 몸체 안에서 직접 변경해도 불변성을 유지한다.
        getProducts: (state) =>{
            state.loading = true;
        },
        getProductsSuccess: (state, { payload }) => {
            state.data = payload;
            state.loading = false;
        },
        getProductsError: (state, {payload })=>{
            state.error = payload;
            state.loading = false;
        }
    }
})
// 정의한 액션과 리듀서를 export한다.
export const productsActions = productsSlice.actions;
// 아래와 같이 actions을 명시하는 것도 가능하다.
//export const {getProducts,getProductsSuccess,getProductsError } = productsSlice.actions;

// reducer의 RootState 타입을 지정
export type ProductState = ReturnType<typeof productsSlice.reducer>;

export default productsSlice;
```
### rootReducer 생성
#### reducers/index.ts
```js
import { combineReducers } from "redux";
import productsSlice from "./productReducer";

const rootReducer = productsSlice.reducer;

export default rootReducer;
```

### sagaApi 생성. 내부에서 Redux Action 호출
#### sagas/sagaApi.ts
```js
import axios, { AxiosResponse } from "axios";
import { all, call, fork, put, takeEvery, takeLatest } from "redux-saga/effects";
import { productsActions } from "../reducers/productReducer";

const CallApi = () => {
  return axios.get("https://dog.ceo/api/breeds/image/random")
}

function* getApiProducts() {
    // yield과정에서 발생하는 error는 catch에서 걸린다.
    try {
      // API 요청을 한다.
      // call(fn)에서 만약 fn이 Promise를 반환한다면 resovle될 때 까지 기다리고 결과를 generate한다.
      // 따라서 response에는 API 요청 응답이 담기게 된다.
      // axios를 통해 요청하기 때문에 AxiosResponse로 타입을 명시해준다.
      const response: AxiosResponse = yield call(CallApi);
      console.log("Response" + response.data)
      // put(action)은 action을 dispatch를 한다.
      yield put(productsActions.getProductsSuccess(response.data));
    } catch (error) {
      // 위 과정에서 에러가 발생하면 여기서 다룬다.
      yield put(productsActions.getProductsError(error));
    }
  }
  
  // 그 다음 getProducts 액션을 감지하는 saga를 작성한다.
  function* watchGetProducts() {
    // 만약 getProducts액션 (패턴이라고도 하는데)이 감지되면, getProductsSaga를 호출한다.
    yield takeLatest(productsActions.getProducts, getApiProducts);
  }
  
  // watchGetProducts를 바로 export 해서 rootSaga에 넣어도 되는데 saga가 여러개 인 경우 saga로 한번더 감싸준다.
  export default function* getApiProductsSaga() {
    yield all([fork(watchGetProducts)]);
  }
```
Api주소는 임의로 셋팅

### saga를 호출할 rootSaga 생성
#### sagas/index.ts
```js
import { all, fork } from 'redux-saga/effects';
import getApiProductsSaga from './sagaApi';

export default function* rootSaga() {
  yield all([fork(getApiProductsSaga)]);
}
```

### reducer와 saga를 nextjs middleware에 적용
#### modules/store.ts
```js
import { createStore, applyMiddleware, compose, Middleware, StoreEnhancer } from "redux";
import { createWrapper, MakeStore } from "next-redux-wrapper";
import { createLogger } from "redux-logger";
import { composeWithDevTools } from "redux-devtools-extension";
import createSagaMiddleware from "redux-saga";
import rootSaga from "./sagas";
import productsSlice from "./reducers/productReducer"
import rootReducer from "./reducers";


// 미들웨어 끼리 묶음
const bindMiddleware = (middlewares: Middleware[]): StoreEnhancer => {
  const logger = createLogger();

  if (process.env.NODE_ENV !== "production") {
    const { composeWithDevTools } = require("redux-devtools-extension");
    return composeWithDevTools(
      applyMiddleware(...middlewares),
      applyMiddleware(logger)
    );
  }
  return compose(applyMiddleware(...middlewares), applyMiddleware(logger));
};

// Next Redux Wrapper 리듀서와 rootSaga를 묶음
export const makeStore = () => {
  const sagaMiddleware = createSagaMiddleware();

  const store = createStore(rootReducer, bindMiddleware([sagaMiddleware]));

  sagaMiddleware.run(rootSaga);

  return store;
};

export const wrapper = createWrapper(makeStore, { debug: true });
```

### 메인에서 Redux 사용을 명시
#### _app.tsx
```js
import { NextPage } from "next";
import { AppProps } from "next/app";
import React from "react";
import { wrapper } from "../modules/store";

const MyApp: NextPage<AppProps> = ({ Component, pageProps }: AppProps) => {
  return <Component {...pageProps} />;
};

export default wrapper.withRedux(MyApp);
```
