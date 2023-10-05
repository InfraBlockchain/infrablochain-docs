---
title: 오프체인 HTTP 요청하기
description: 오프체인 워커를 사용하여 HTTP 요청을 하는 방법을 설명합니다.
keywords:
  - 오프체인 워커
  - OCW
  - HTTP
  - HTTPS
  - 요청

대부분의 블록체인은 자체 네트워크 외부에 호스팅된 데이터에 접근할 수 없기 때문에, 일반적으로 외부 제3자 서비스인 **오라클**을 사용하여 네트워크 외부의 위치에서 정보를 가져오거나 정보를 전송합니다.
Substrate 기반 블록체인에서는 **오프체인 워커**(OCW)가 이와 유사한 기능을 제공하지만, 온체인 상태에 접근할 수 있는 장점이 있습니다.

이 가이드에서는 오프체인 워커를 사용하여 GET 또는 POST 메서드를 사용하여 HTTP 요청을 하는 방법을 설명합니다.
이 가이드의 예제에서는 `cryptocompare` API에서 비트코인 가격을 가져오는 방법과 오프체인 워커 API를 사용하여 데이터를 제출하는 방법을 볼 수 있습니다.

Rust는 HTTP 요청을 보내기 위한 자체 라이브러리를 제공한다는 것을 알 수 있습니다.
그러나 오프체인 워커는 자체 WebAssembly 실행 환경인 [no-std](https://docs.rust-embedded.org/book/intro/no-std.html) 환경에서 실행되므로 표준 Rust 라이브러리에 액세스할 수 없습니다.
대신, Substrate는 HTTP 요청을 보내기 위해 사용할 수 있는 자체 라이브러리를 제공합니다.

Substrate HTTP 라이브러리는 다음과 같은 메서드를 지원합니다:

- GET
- POST
- PUT
- PATCH
- DELETE

## 데드라인 설정 및 HTTP 요청 인스턴스화

대부분의 경우, 오프체인 워커가 작업을 실행하는 데 허용되는 시간을 제한하고 싶을 것입니다.
이 예제에서는 외부 호출을 완료하기 위해 2초의 하드 코딩된 데드라인을 설정할 수 있습니다.
또한 응답을 무기한으로 기다릴 수도 있습니다.
그러나 무기한으로 기다리는 것은 외부 호스트 머신에서 타임아웃이 발생할 수 있습니다.

1. 2초의 데드라인을 생성합니다.

   ```rust
   let deadline = sp_io::offchain::timestamp().add(Duration::from_millis(2_000));
   ```

1. 외부 HTTP GET 요청을 시작합니다.

   ```rust
   let request = http::Request::get("https://min-api.cryptocompare.com/data/price?fsym=BTC&tsyms=USD");
   let pending = request.deadline(deadline).send().map_err(|_| http::Error::IoError)?;
   let response = pending.try_wait(deadline).map_err(|_| http::Error::DeadlineReached)??;
   ```

## 응답 읽기 및 제출하기

1. 응답 상태 코드를 확인합니다.

   ```rust
   // 응답을 읽기 전에 상태 코드를 확인해 봅시다.
   if response.code != 200 {
     log::warn!("예상하지 못한 상태 코드: {}", response.code);
     return Err(http::Error::Unknown)
   }
   ```

1. 응답을 읽습니다.

   ```rust
   let body = response.body().collect::<Vec<u8>>();
   // 바디에서 str 슬라이스를 생성합니다.
   let body_str = sp_std::str::from_utf8(&body).map_err(|_| {
     log::warn!("UTF8 바디 없음");
     http::Error::Unknown
   })?;
   ```

1. POST 요청을 사용하여 API에 데이터를 제출합니다.

   ```rust
    // POST 요청 보내기
   let request_body = Vec::new();
   let request = http::Request::post("https://reqres.in/api/users", vec![request_body.clone()])
     .add_header("x-api-key", "test_api_key")
     .add_header("content-type", "application/json");

   let pending = request
     .deadline(deadline)
     .body(vec![request_body.clone()])
     .send()
     .map_err(|_| http::Error::IoError)?;

   // 응답 기다리기
   let response = pending
     .try_wait(deadline)
     .map_err(|_| http::Error::DeadlineReached)??;
   ```

## 예제

- [예제 팔레트: 오프체인 워커](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs)
- [데모: OCW 팔레트](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/master/pallets/ocw/src/lib.rs#L363-#L401)
- [소스: Substrate 코어 프리미티브](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/primitives/runtime/src/offchain/http.rs#L63-L76)