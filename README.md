### Version 1

```swift
import Foundation
import Combine

// MARK: - VERSION 1 FOR EASIER UNDERSTANDING

struct User: Decodable {
    let id: UUID
}

struct UserDetails: Decodable {
    let name: String
    let email: String
}

struct Friend: Decodable {
    let id: UUID
    let name: String
}

func loadUser() -> AnyPublisher<User, Error> {
    let url = URL(string: "https://a-user.com")!
    return URLSession.shared.dataTaskPublisher(for: url)
        .tryMap { result in
            guard let httpResponse = result.response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
            return result.data
        }
        .decode(type: User.self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

func loadDetails(user: User) -> AnyPublisher<UserDetails, Error> {
    let url = URL(string: "https://a-user.com/\(user.id)/details")!
    return URLSession.shared.dataTaskPublisher(for: url)
        .tryMap { result in
            guard let httpResponse = result.response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
            return result.data
        }
        .decode(type: UserDetails.self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

func loadFriends(user: User) -> AnyPublisher<[Friend], Error> {
    let url = URL(string: "https://a-user.com/\(user.id)/friends")!
    return URLSession.shared.dataTaskPublisher(for: url)
        .tryMap { result in
            guard let httpResponse = result.response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
            return result.data
        }
        .decode(type: [Friend].self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

// Dependent Network Call
func loadUserDetails() -> AnyPublisher<UserDetails, Error> {
    loadUser()
        .flatMap(loadDetails)
        .eraseToAnyPublisher()
}

// Dependent Network Call
func loadUserDetailsAndFriends() -> AnyPublisher<(UserDetails, [Friend]), Error> {
    return loadUser()
        .flatMap({ user in
            Publishers.Zip(loadDetails(user: user), loadFriends(user: user))
        })
        .eraseToAnyPublisher()
}
```

### Version 2

```swift
import Foundation
import Combine

// MARK: - VERSION 2 - REFACTORED
struct User: Decodable {
    let id: UUID
}

struct UserDetails: Decodable {
    let name: String
    let email: String
}

struct Friend: Decodable {
    let id: UUID
    let name: String
}

func loadUser() -> AnyPublisher<User, Error> {
    let url = URL(string: "https://a-user.com")!
    return load(url: url)
}

func loadDetails(user: User) -> AnyPublisher<UserDetails, Error> {
    let url = URL(string: "https://a-user.com/\(user.id)/details")!
    return load(url: url)
}

func loadFriends(user: User) -> AnyPublisher<[Friend], Error> {
    let url = URL(string: "https://a-user.com/\(user.id)/friends")!
    return load(url: url)
}

func load<T: Decodable>(url: URL) -> AnyPublisher<T, Error> {
    return URLSession.shared.dataTaskPublisher(for: url)
        .tryMap { result in
            guard let httpResponse = result.response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw URLError(.badServerResponse)
            }
            return result.data
        }
        .decode(type: T.self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

// Dependent Network Call
func loadUserDetails() -> AnyPublisher<UserDetails, Error> {
    loadUser()
        .flatMap(loadDetails)
        .eraseToAnyPublisher()
}

// Dependent Network Call
func loadUserDetailsAndFriends() -> AnyPublisher<(UserDetails, [Friend]), Error> {
    return loadUser()
        .flatMap({ user in
            Publishers.Zip(loadDetails(user: user), loadFriends(user: user))
        })
        .eraseToAnyPublisher()
}
```
