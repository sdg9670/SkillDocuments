# Vue.js에 slide 적용하기 (Vueper Slides)

## 시작 및 사용법

**모듈 설치**

```bash
npm install --save vuperslides
```

**Import**

```javascript
import { VueperSlides, VueperSlide } from 'vueperslides'
import 'vueperslides/dist/vueperslides.css'
...

export default {
  components: { VueperSlides, VueperSlide },
  ...
}
```

> 모듈을 설치하고 임포트를 하면 사용할 준비가 끝납니다.  
> 사용법은 해당 모듈 사이트에 너무 자세히 나와있어 부가 설명이 불필요할 것으로 판단하여 사이트 주소를 남깁니다.
> 공식 사이트: [`Vueper Slides`](https://antoniandre.github.io/vueper-slides/)
>  
> 하단 부분에는 회원가입을 예로든 예제가 있습니다. 
>  
> **간단하게 만들어서 코드는 볼 것이 없지만 `여러가지 옵션을 이용한 커스텀마이징`이 가능하다는 점을 꼭 기억해두시면 좋을 것 같습니다. 슬라이더를 이미지 외에도 사용할 곳이 많다는 점을 알려드리고 싶었습니다.**
>
> *주의 사항으로는 slide 컴포넌트 내부에는 key값을 설정해야지 오류가 안납니다.*

## 예제

**코드**

```html
<template>
  <div class="out" style="text-align: center; vertical-align: middle">
    <vueper-slides class="in" :touchable="false" :bullets="false" :arrows="false" ref="first" style="width: 500px; height: 600px; display: inline-block;">
      <vueper-slide
        :key="1"
        :style="'background-color: #42b983'">
        <template v-slot:content>
          <div>
            ID: <input type="text"/><br/>
            PW: <input type="text"/><br/>
            <button @click="$refs.first.next()">다음</button>
          </div>
        </template>
      </vueper-slide>
      <vueper-slide
        :key="2"
        :style="'background-color: #42b983'">
        <template v-slot:content>
          <div>
            나이: <input type="text"/><br/>
            주소: <input type="text"/><br/>
            <button @click="$refs.first.previous()">이전</button>
            <button @click="$refs.first.next()">다음</button>
          </div>
        </template>
      </vueper-slide>
      <vueper-slide
        :key="3"
        :style="'background-color: #42b983'">
        <template v-slot:content>
          <div>
            자기소개: <textarea/><br/>
            <button @click="$refs.first.previous()">이전</button>
            <button v-on:click="showAlert('가입을 축하드립니다')">가입완료</button>
          </div>
        </template>
      </vueper-slide>
    </vueper-slides>
  </div>
</template>

<script>
import { VueperSlides, VueperSlide } from 'vueperslides'
import 'vueperslides/dist/vueperslides.css'
export default {
  components: { VueperSlides, VueperSlide },
  methods: {
    showAlert(msg) {
      alert(msg);
    }
  }
}
</script>
```

**결과**

![결과](./VueperSlides/image1.gif)