---
layout: post
title: 네트워크 통신에 Generic을 적용 후기
author: piorosen
tags: [generic, typealias, swift]
hide_title: false
categories: [Blogging, Develop]
# feature-img: "assets/img/feature-img/"
# feature-img: "http://placehold.it/700x300"
# thumbnail: "assets/img/thumbnails/feature-img/"
# thumbnail: "http://placehold.it/200x200"
---

## 개요
연구과제를 진행 하던 중 클라이언트와 네트워크 통신 개발을 맡게 되었다. 개발을 진행 하면서 "Request" 를 하게 되면 반드시 "Acknowledge"가 반환이 되는 구조 였다.
현재 개발 진행 내용에서는 클라이언트에서 서버로 요청을 하게 되면 서버에서 처리를 한 뒤 클라이언트로 보내주는 방식으로만 되어 있기 때문에(즉 서버에서 클라이언트로 요청하는 것이 없음)
Request 객체에 Acknowledge 타입이 있다면 해당 라이브러리를 사용 하는 사람은 조금 더 쉽게 사용할 수 있는 구조가 되지 않을까? 라는 생각으로 시작하게 되었다.

---

## Prerequisite

- Swift 언어로 작성이 되어 있으므로, Swift에 대한 문법에 이해 하고 있으면 좋습니다.
- Generic과 Interface(Protocol), Inheritance(상속)에 대한 내용이 있으므로 필수적입니다.

---

## 코드 개요

Class나 Struct는 기본적으로 최소 5개로 이뤄져 있으며 더 줄인다면 줄일 수 있겠지만 현재 작업을 하였을 때 5개로 이뤄져 있습니다.

- NetworkManager
- RequestMessage
- AcknowledgeMessage
- ex) RequestLogin
- ex) AcknowledgeLogin

네트워크 통신을 하기 위한 객체와 받아서 처리를 할 ReuqestMessage와 그에 대한 응답을 위한 AcknowledgeMessage가 존재 합니다.

---

## 기본 골격 설명

기본적인 구조는 RequestMessage가 있으며, Request에 대한 서버 응답은 ACK_MESSAGE란 타입으로 결과가 반환이 된다.
RequestMessage.url 같은 경우에는 네트워크 통신을 위한 변수 이므로, Property의 Get으로 구현을 하였으며 static이 아닌 이유는 Request 변수에 따라서 url이 변경이 되는 경우가 있어 유동적으로 변경이 가능하게 위함이다.
makeParameter은 JSON 이나, XML 등 다양한 포맷을 지원 하기 위함이긴 하나, 연구과제 서버가 Json이나, POST의 파라미터 등등 다양한 포맷을 동시에 사용하게 되면서 Dictionary로 일단 반환을 하고, NetworkManager에서 처리를 하도록 하였다.

```swift
public protocol RequestMessage {
    // Generic임 C#에서는 RequestMessage<ACK_MESSAGE> 와 동일
    associatedtype ACK_MESSAGE: AcknowledgeMessage

    // 네트워크 통신을 위한 RestAPI 주소
    var url: String { get }

    // 통신 파라미터를 만들기 위한 딕셔너리
    func makeParameters() -> [String:String]
}

public protocol AcknowledgeMessage {
    // static 클래스 이며 decode를 하게 될 경우 자기 자신의 타입, 생성자로 쓰인다.
    static func decode(dgram: Data) -> Self
}
```

구조 자체는 많이 간단하다. RequestMessage에 AcknowledgeMessage 만 타입 지정이 가능하도록 하여 상속 받아 개발을 할 때 어떤 타입으로 제공을 할지 코드 상에서 정해지게 되다 보니 프로그램을 실행 해서 테스트가 아닌 코드 레벨에서 오류를 찾아 해결을 할 수 있는 장점이 있었다. (즉 런타임이 아닌 컴파일 타임 때 오류를 잡을 수 있다.)

오류를 런타임이 아닌 컴파일 타임에 잡을 수 있다는건 정말 좋은 의미이다. Unit Test 코드를 작성을 하지 않고 코드만 보고 판단이 가능 하다는건 리팩토링이나 코드 리뷰 할 때 검수를 한번 더 할 수 있다는 의미 이기도 하며, 유지보수에 좋기 때문이다.

```swift
public class NetworkManager {
    public func request<T: RequestMessage>(_ request: T, _ callback: @escaping (T.ACK_MESSAGE) -> Void) {
        var request = URLRequest(url: URL(string: request.url)!)
        request.httpMethod = "POST"
        request.httpBody = JSON(req.makeParameters()).rawString() ?? String()

        URLSession.shared.dataTask(with: request, completionHandler: { d, res, e in
            guard let data = d else {
                if let error = e {
                    print(error)
                }
                return
            }

            let result = T.ACK_MESSAGE.decode(dgram: data)
            callback(result)
        }).resume()
    }
}
```

통신을 담당하는 NetworkManager는 마치 Strategy Pattern 과 같이 구현에 따라서 모든 결과가 달라지게 된다.

만약 아래 처럼 Generic으로 받지 않고 하나의 타입으로 바로 받는다면 문제가 발생 하게 된다.

```swift
func request<T: RequestMessage>(_ request: T, _ callback: @escaping (T.ACK_MESSAGE) -> Void)

func request(_ request: ReuqestMessage, _ callback: @escaping (/* 여기에는 어떤 값으로 정의가 되어야 하는가? */.ACK_MESSAGE) -> Void)
```

이처럼 반환형의 타입이 정해지지 않게 되므로 Generic으로 구현해야 했다. Generic으로 구현하여 함수를 호출 할 때, 반환 타입을 만들어 반환 하도록 하였다.
여기서 RequestMessage를 상속 받은 클래스를 구현 한 뒤 NetworkManager.request에 호출 하면 동작 한다.

```swift
// Request Login의 에시, (RequestMessage를 상속 받음)
public struct RequestLogin : RequestMessage {
    public init(identify: String, password: String) {
        self.identify = identify
        self.password = password
    }

// 반환 타입
    public typealias ACK_MESSAGE = AcknowledgeLogin

// 서버 호스트와 API 요청지
    public let url = "http://localhost/login"

// API 요청 파라미터
    public var identify: String
    public var password: String

// NetworkManager에서 필요한 Key-Value 형태의 딕셔너리
    public func makeParameters() -> [String:String] {
        var data = [String:String]()

        data["identi"] = identify
        data["password"] = password

        return data
    }
}

// Request Login의 반응 타입
public struct AcknowledgeLogin : AcknowledgeMessage {
// 로그인 성공 / 실패 유무
    public let success: Bool

// Data == Byte[] 타입이 넘어 왔을 때 디코딩 함수
    public static func decode(dgram: Data) -> AcknowledgeLogin {
        let result: BCAckLogin
        if let text = String(data: dgram, encoding: .utf8), let r = Bool(text) {
            return AcknowledgeLogin(success: r)
        }

        return AcknowledgeLogin(success: false)
    }

}
```

실제 코드 구현으로는 위 처럼 RequestMessage를 상속 받아 서버 API 요청지와 요청 파라미터를 정의하여 구현 하고, Acknowledge부분은 서버로 부터 응답을 받아 디코딩 하여 ACK_MESSAGE 타입으로 반환 한다. 실제 사용은 매우 간편하고, 직관적으로 동작한다.

```swift
let message = RequestLogin(identify: "testUser", password: "password")
NetworkManager().reqeust(message) { ack in
    print(ack)
}
/* 다른 언어 경우 (C#) 형식으로 할 경우
var message = RequestLogin("testuser", "password");
NetworkManager().request(message, (ack) => {
    Console.WriteLine(ack);
})
*/
```

## 후기

연구과제를 진행 하면서 기존에 완성이 되어 있던 서버에 대응을 해야 했었다. 그렇지만 서버쪽 API 양 자체가 30개가 넘어 일일이 하나씩 구현하기에는 많은 양이고, 많은 노가다가 필요한 작업이 될것이 뻔하였다.
안드로이드 코드랑 똑같이 코드를 작성 하는 것이 아닌, 코드를 리팩토링하면서 최적화를 해보자 라는 생각을 하게 되었다. 코드를 여러번 지우고, 다시 쓰는 작업을 하게 되었는데 그 중 최고의 방법이 Request 타입에서 Acknowledge타입이 확정이 된다면 얼마나 좋을까? 라는 생각으로 시작이 되어서 Generic을 이용하면 될 것 같아 시작 하게 되었다.
