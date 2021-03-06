# LSTM

앞서 배운 RNN은 조금 더 정확히 따지자면 `Vanilla RNN`이라고 부릅니다.

이 바닐라 RNN의 단점을 보완한 것이 LSTM 입니다.

## Vanilla RNN의 단점

RNN은 이전의 계산이 결과에 영향을 주는 형태였습니다. 하지만 이는 바닐라 RNN의 time step이 길어질 수록 앞의 정보가 뒤로 충분히 전달되지 못합니다.

![030-1](https://user-images.githubusercontent.com/63298243/105702807-aa59d080-5f4f-11eb-8576-eff1ddad4672.png)

위의 그림에서 X<sub>1</sub>의 데이터 양을 진한 파란색으로 표현했을때, time step가 증가할수록 파란색이 옅어지는 것을 볼 수 있습니다.

이는 X<sub>1</sub>의 데이터가 손실되는 것을 의미합니다. X<sub>t</sub>의 시점에서는 X<sub>1</sub>의 데이터는 결과에 영향을 주지 않을지도 모릅니다.

즉 은닉층의 과거의 정보가 마지막까지 전달되지 못하는 현상이 발생한다는 것이고, 이를 **장기 의존성 문제(the problem of Long-Term Dependencies)**라고 합니다.

## LSTM

![030-2](https://user-images.githubusercontent.com/63298243/105702810-ac239400-5f4f-11eb-99a1-493b8a1b00d9.png)

LSTM은 Long Short-Term Memory(장단기 메모리)라고 합니다.

은닉층의 메모리 셀에 입력 게이트, 망각 게이트, 출력 게이트를 추가하여 불필요한 기억을 지우고, 기억해야할 것들을 저장합니다.

LSTM은 Vanilla RNN보다 비교적 긴 시퀀스(time step)를 처리하는데 뛰어난 성능을 가집니다.


### 입력 게이트

![030-3](https://user-images.githubusercontent.com/63298243/105702815-aded5780-5f4f-11eb-9472-9cfb46b8d3d0.png)

i<sub>t</sub>=σ(W<sub>xi</sub>x<sub>t</sub>+W<sub>hi</sub>h<sub>t-1</sub>+b<sub>i</sub>)  
g<sub>t</sub>=tanh(W<sub>xg</sub>x<sub>t</sub>+W<sub>hg</sub>h<sub>t-1</sub>+b<sub>g</sub>)

입력 게이트는 현재 정보를 기억하기 위한 게이트입니다. 우선 현재 시점 t의 x값과 입력 게이트로 이어지는 가중치 Wxi를 곱한 값과 이전 시점 t-1의 은닉 상태가 입력 게이트로 이어지는 가중치 W<sub>hi</sub>를 곱한 값을 더하여 시그모이드 함수를 지납니다. 이를 i<sub>t</sub>라고 합니다.

그리고 현재 시점 t의 x값과 입력 게이트로 이어지는 가중치 W<sub>xi</sub>를 곱한 값과 이전 시점 t-1의 은닉 상태가 입력 게이트로 이어지는 가중치 W<sub>hg</sub>를 곱한 값을 더하여 하이퍼볼릭탄젠트 함수를 지납니다. 이를 g<sub>t</sub>라고 합니다.

시그모이드 함수를 지나 0과 1 사이의 값과 하이퍼볼릭탄젠트 함수를 지나 -1과 1사이의 값 두 개가 나오게 됩니다. 이 두 개의 값에 따라 기억할 정보의 양을 정하게 됩니다.

### 삭제 게이트

![030-4](https://user-images.githubusercontent.com/63298243/105702817-ae85ee00-5f4f-11eb-8905-4ceb6b73ea7e.png)

f<sub>t</sub>=σ(W<sub>xf</sub>x<sub>t</sub>+W<sub>hf</sub>h<sub>t-1</sub>+b<sub>f</sub>)

삭제 게이트는 기억을 삭제하기 위한 게이트입니다. 

현재 시점 t의 x값과 이전 시점 t-1의 은닉 상태가 시그모이드 함수를 지나게 됩니다. 

시그모이드 함수를 지나면 0과 1 사이의 값이 나오게 되는데, 이 값이 곧 삭제 과정을 거친 정보의 양입니다. 

0에 가까울수록 정보가 많이 삭제된 것이고 1에 가까울수록 정보를 온전히 기억한 것입니다.

### 셀 상태 (장기 상태)

![030-5](https://user-images.githubusercontent.com/63298243/105702819-ae85ee00-5f4f-11eb-9aeb-4c7de2adce16.png)

C<sub>t</sub>=f<sub>t</sub>∘C<sub>t-1</sub>+i<sub>t</sub>∘g<sub>t</sub>

셀 상태는 위의 그림에서 왼쪽에서 오른쪽으로 가는 굵은 선입니다. 셀 상태 또한 은닉 상태처럼 이전 시점의 셀 상태가 다음 시점의 셀 상태를 구하기 위한 입력으로서 사용됩니다.

셀 상태 Ct를 LSTM에서는 장기 상태라고 부르기도 합니다. 그렇다면 셀 상태를 구하는 방법을 알아보겠습니다. 

앞의 연산을 통해 삭제 게이트에서 일부 기억을 잃은 상태입니다.

입력 게이트에서 구한 i<sub>t</sub>, g<sub>t</sub> 이 두 개의 값에 대해서 원소별 곱(entrywise product)을 진행합니다. 

다시 말해 같은 크기의 두 행렬이 있을 때 같은 위치의 성분끼리 곱하는 것을 말합니다. 여기서는 식으로 ∘ 로 표현합니다. 이것이 이번에 선택된 기억할 값입니다.

입력 게이트에서 선택된 기억을 삭제 게이트의 결과값과 더합니다. 

이 값을 현재 시점 t의 셀 상태라고 하며, 이 값은 다음 t+1 시점의 LSTM 셀로 넘겨집니다.

### 출력 게이트 (단기 상태)

![030-6](https://user-images.githubusercontent.com/63298243/105702822-af1e8480-5f4f-11eb-8434-32b9d310a115.png)

o<sub>t</sub>=σ(W<sub>xo</sub>x<sub>t</sub>+W<sub>ho</sub>h<sub>t-1</sub>+b<sub>o</sub>)
h<sub>t</sub>=o<sub>t</sub>∘tanh(c<sub>t</sub>)

출력 게이트는 현재 시점 t의 x값과 이전 시점 t-1의 은닉 상태가 시그모이드 함수를 지난 값입니다. 

이 값은 현재 시점 t의 은닉 상태를 결정합니다.

은닉 상태를 단기 상태라고도 합니다. 

장기 상태의 값이 하이퍼볼릭탄젠트 함수를 지나 -1과 1사이의 값이 되고, 이 값은 출력 게이트의 값과 연산되어 은닉 상태(단기 상태)가 됩니다. 