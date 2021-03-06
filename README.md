본 Repository는 Cyark사이트를 모방하여 참고용으로 제작되었음을 알립니다.

문제가 될 경우 바로 삭제하겠습니다.

# Google Maps API

1. 구글 맵 API를 활용하여 정보를 표시하고 클러스터링 및 다양한 기능에 대해 알아봄
2. 마커를 클릭할 때 모달창을 표시하며 다양한 기능과 상호작용 할 수 있도록 코딩

## 구글 맵 API 연결하기

구글 API 연결에 필요한 script

```html
<!-- 구글 맵 -->
<script
  async
  src="https://maps.googleapis.com/maps/api/js?key=(YOUR API KEY)&callback=initMap"
></script>
```

로 연결 가능

API Key의 경우에는 간단한 검색으로 회원가입 이후 바로 발급받을 수 있으니 구글검색을 참고

연결하는 과정은 매우 간단하므로 거두절미 하고 본격적인 기능설명

## 상단 컨텐츠 표시 제어 영역

<img src="gitImages\Top_Age.png">

지도 상단에 위와같은 바 영역이 존재함

클릭 및 드래그로 좌 우 영역 조절이 가능하며 영역밖으로 벗어난 연도의 컨텐츠의 경우에는

화면에 표시되지 않음

```javascript
// 화면 상단 드래그 이벤트
let LeftDragging = false;
let RightDragging = false;
window.addEventListener("mousedown", (e) => {
  switch (e.target) {
    case LeftHandle:
    case LeftIcon:
      LeftDragging = true;
      break;
    case RightHandle:
    case RightIcon:
      RightDragging = true;
      break;
    default:
      break;
  }
});
// 드래그 후 이동
window.addEventListener("mousemove", (e) => {
  if (LeftDragging && e.x > -1) {
    LeftHandle.style.left = e.x + "px";
  }
  if (RightDragging && e.x + RightHandle.offsetWidth < window.innerWidth) {
    RightHandle.style.left = e.x + "px";
  }
});
// 드래그 후 커서를 뗀다
window.addEventListener("mouseup", (e) => {
  LeftDragging = false;
  RightDragging = false;
});

// 화면 사이즈 조정 시 상단 바 최대위치로
const body = document.querySelector("body");
window.addEventListener("resize", (e) => {
  RightHandle.style.left =
    e.currentTarget.innerWidth - RightHandle.offsetWidth + "px";
  LeftHandle.style.left = 0;
});

// 화면 상단 클릭 이벤트
document.querySelector(".FilterBar").addEventListener("click", (e) => {
  const Top_GoldenBars = document.querySelectorAll(".Top_GoldenBar");
  if (e.x + 17 < window.innerWidth) {
    let Right_Blank = RightHandle.offsetLeft;
    let Left_Blank = LeftHandle.offsetLeft;
    if (e.x - Left_Blank <= Right_Blank - e.x) {
      LeftHandle.style.left = e.x + "px";
    } else {
      RightHandle.style.left = e.x + "px";
    }
  }
  HandleDistance.style.left = `${
    LeftHandle.getBoundingClientRect().x + LeftHandle.offsetWidth
  }px`;
  HandleDistance.style.width = `${
    RightHandle.getBoundingClientRect().x -
    (LeftHandle.getBoundingClientRect().x + LeftHandle.offsetWidth)
  }px`;
  let FiltedArray = Array.from(Top_GoldenBars)
    .filter(
      (a) =>
        Math.round(a.getBoundingClientRect().x) >
        LeftHandle.getBoundingClientRect().x + LeftHandle.offsetWidth
    )
    .filter(
      (a) =>
        Math.round(a.getBoundingClientRect().x) <
        RightHandle.getBoundingClientRect().x
    )
    .map((a) =>
      a.innerText.includes("BCE")
        ? Number("-" + a.innerText.match(/[0-9]/g).join(""))
        : Number(a.innerText.match(/[0-9]/g).join(""))
    );
  MinNumber = FiltedArray[0];
  MaxNumber = Math.max.apply(null, FiltedArray);
  markers.map((a) => a.setMap(map));
  markers
    .filter((a) => a.history < MinNumber || a.history > MaxNumber)
    .map((a) => a.setMap(null));
});
```

window객체에 바로 리스너를 연결하였으며 클릭했을 때, 클릭한 후 이동, 클릭을 멈췄을 때

로 나뉘며 이벤트를 수행함

## 마커 표시

<img src="gitImages\Marker.png">

지도를 호출하는 것 뿐 아니라 Google Maps API를 쓰는 궁극적인 목표는 자신이 원하는

지점에 상호작용을 용이하게 관리하는 것에 있음

```javascript
// 아래에 오는 모든 코드는 맵 출력과 관련이 있음
async function initMap() {

    // 맵 (뼈대)
    map = new google.maps.Map(document.getElementById("map"), {
        center: marker,
        zoom: 12,
        // 길어서 생략
        styles: [...]
    });

    // 마커 렌더링
    const markers = locations.map((location, index) => new google.maps.Marker({
        position: location,
        label: 'A',
        history: location.history
    }))
}
```

위와 같이 map이라는 변수에 생성자를 넣어 맵을 구현할 수 있으며 이 경우에는 무조건

async function initMap함수 안에 존해하여야 함

마커 또한 마커 생성자가 maps객체 안에 존재하며 lon,lat실제 좌표를 지정하여 코딩할 경우

해당 위치에 표시됨

## 클러스터링

<img src="gitImages\Clustering.png">

클러스터링 이란 A라는 한 지점에 여러 포인트가 겹쳐 있을 경우 멀리서 봤을 때

지저분해 보일 수 있기 때문에 하나의 마커로 합쳐 보이게 도와줌

위의 이미지는 클러스터링에 이용할 사진 경로를 설정해주지 않아서 깨진 것이며,

각각 15개의 마커, 3개의 마커, 5개의 마커가 해당 위치에 집결돼있음을 알려줌

```html
<!-- 마커 클러스터링 -->
<script src="https://unpkg.com/@google/markerclustererplus@4.0.1/dist/markerclustererplus.min.js"></script>
```

위의 cdn을 추가해주어야 정상적인 이용이 가능함

```javascript
// 모여있는 마커 클러스터링
new MarkerClusterer(map, markers, {
  // 해당 (경로)1.png (경로)2.png 등으로 불러오기 때문에
  // 이름을 적절하게 변경해야 함
  imagePath: "textures/TrexTooth",
  // 해당 경우에는 TrexTooth1.png 가 가장 많은 마커가 모여있을 때 의 이미지로 사용됨
});
```

## 마커 클릭 시 이벤트 창

<img src="gitImages\Modal.png">

마커를 클릭 한 경우 내가 필요한 정보를 사용자에게 표시해주는 방법이 있음

```javascript
// infoWindow 배출함수
function InfoFactory(country, name, img) {
  return `
        <div style="background-image:url('${img[0]}')" class="clickImg">
            <div class="BtmBlack">
                
            </div>
            <div class="btmText">
                <div class="titleCountry">${country}</div>
                <div class="titleName">${name}</div>
            </div>
            <div class="RightBtmBtn">
                <div class="CircleLine">
                    ${img.map((a, i) => {
                      if (i === 0) {
                        return `<div onclick="this.parentNode.parentNode.parentNode.style.backgroundImage = 'url(${img[0]})';Array.from(this.parentNode.childNodes).filter(a => a instanceof HTMLElement === true ).map(a => {a.classList.remove('Selected'); this.classList.add('Selected')})" class="Circle Selected"></div>`;
                      } else {
                        return `<div onclick="this.parentNode.parentNode.parentNode.style.backgroundImage = 'url(${img[i]})';Array.from(this.parentNode.childNodes).filter(a => a instanceof HTMLElement === true ).map(a => {a.classList.remove('Selected'); this.classList.add('Selected')})" class="Circle"></div>`;
                      }
                    })}
                </div>
            </div>
            <div class="btmLine"></div>
        </div>
    `;
}

// 마커1 정보
const pointInfo = new google.maps.InfoWindow({
  content: InfoFactory("Korea", "BaekDooSan", [
    "https://github.com/kwb020312/ArcGIS_KeyongCheonSazi/blob/master/gitImages/Public_Image.png?raw=true",
    "https://github.com/kwb020312/ArcGIS_KeyongCheonSazi/blob/master/gitImages/Complete.png?raw=true",
  ]),
});

// 마커1 클릭 이벤트
markers[0].addListener("click", () => {
  pointInfo.open(map, markers[0]);
});
```

위와 같이 나의 경우에는 생성자 함수를 만들어 간편하게 다른 마커에도 적용할 수 있도록

적용하였으며 마커의 경우에는 마커.addEventListener() 가 아닌 addListener() 함수

라는 것을 알아두어야 한다.

## 프리뷰

<img src="gitImages\Preview.png">

현재의 경우에는 API결제 키를 빼두었기 때문에 위와 같이 결제 후 이용해달라는

경고 메시지가 뜸

API 결제 후 이용한다면 위 이미지와 같은 보기 싫은 상황의 접근을 막을 수 있음
