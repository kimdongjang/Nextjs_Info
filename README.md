# Nextjs_Info
Next js 정리  
https://kyounghwan01.github.io/blog/React/next/getInitialProps/#getinitialprops-%E1%84%8B%E1%85%B5%E1%84%8C%E1%85%A5%E1%86%B7   
redux https://github.com/kirill-konshin/next-redux-wrapper  
redux+ts https://jktech.tistory.com/46  
redux+codepen https://codesandbox.io/s/next-redux-wrapper-demo-7n2t5?file=/components/store.tsx:230-239  
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
export default class MyApp extends App {

	static async getInitialProps({ Component, ctx }) {
		let pageProps = {};
    
    // 실행하고자 하는 component에 getInitialprops가 있으면 실행하여 props를 받아올 수 있다.
		if (Component.getInitialProps) {
			pageProps = await Component.getInitialProps(ctx);
		}

		return {
			pageProps
		};
	}

	render() {
		const { Component, pageProps, router } = this.props;
    
		return (
			<div>
				<Component {...pageProps} />
			</div>

		);
	}
};
```  
ctx Object의 기본 구성
+ ```pathname``` - 현재 pathname /user?type=normal page 접속 시에는 /user
+ ```query``` - 현재 query를 object형태로 출력 /user?type=normal page 접속 시에는 {type: 'normal'}
+ ```asPath``` - 전체 path /user?type=normal page 접속 시에는 /user?type=normal
+ ```req``` - HTTP request object (server only)
+ ```res``` - HTTP response object (server only)
+ ```err``` - Error object if any error is encountered during the rendering


### getStaticProps
외부데이터를 받아 Static Generation 하기 위한 용도다.
  
```js
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
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  // By returning { props: posts }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts
    }
  };
}

export default Blog;
```
fetch를 통해 게시물을 가져오고 그 게시물의 title을 보여줌

### getStaticPatch
빌드 타임 때, 정적으로 렌더링할 경로 설정  
이곳에 정의하지 않은 하위 경로는 접근해도 페이지가 안뜸  
동적라우팅 : 라우팅 되는 경우의 수 따져서 하위로 넣음  



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

  
