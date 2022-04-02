# Nextjs_Info
Next js 정리


# Next.js
1. pages/ 폴더 안에 다른 폴더를 만들면 해당 라우팅의 페이지들은 서버측에서 먼저 로드해줌
  


## ServerSide Cycle
```js
function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
 
```
1. 이곳에서 렌더링 하는 값은 모든 페이지에 영향을 줍니다.\
2. 최초로 실행되는 것은 _app.tsx
3. _app.tsx은 클라이언트에서 띄우길 바라는 전체 컴포넌트 → 공통 레이아웃임으로 최초 실행되며 내부에 컴포넌트들을 실행함
4. 내부에 컴포넌트가 있다면 전부 실행하고 html의 body로 구성
5. Component, pageProps를 받습니다.
6. 여기서 props로 받은 Component는 요청한 페이지입니다. GET / 요청을 보냈다면, Component 에는 /pages/index.js 파일이 props로 내려오게 됩니다.
7. pageProps는 페이지 getInitialProps를 통해 내려 받은 props들을 말합니다.
8. 그 다음 _document.tsx가 실행됨
9. 페이지를 업데이트 하기 전에 원하는 방식으로 페이지를 업데이트 하는 통로 
10. 

#### 공통적인 데이터 패칭이 필요할 경우 _app.tsx에서 미리 데이터 패칭을 해줌
  

## getInitialProps
#### next v9 이상에서는 getInitialProps 대신 getStaticProps, getStaticPaths, getServerSideProps을 사용

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

  
  
  
