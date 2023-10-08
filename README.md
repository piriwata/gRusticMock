# gRusticMock

`gRusticMock` provides gRPC mocking to help you test Rust applications that interact with third-party libraries.

You can concisely describe the request and response expected in your test cases.

## Table of Contents

- How to install
- Usage
  - Unary Method
- Test isolation
- Efficiency
- References

## How to install

Add `gRusticMock` to your development dependencies by editing the `Cargo.toml` file:

```toml
[dev-dependencies]
# ...
gRusticMock = "0.1"
```

Or by running:
```shell
cargo add wiremock --dev
```

## Usage

### Unary Method

```rust
use grusticmock::{MockServer, Mock, ResponseTemplate};
use grusticmock::matchers::{method, path};

#[async_std::main]
async fn main() {
    // Start a background HTTP server on a random local port
    let mut mock_server = MockServer::start().await;

    let expected := Item { "id": 42, "name": "Item 42" }

    // Arrange the behaviour of the MockServer:
    mock_server
        .expect_unary("myservice/getItems")
        .with_header("locale", "en-US")
        .with_payload(AnyRequest {"id": 42})
        .return(&expected);
    
    // ToDo: Fix me to real code
    let response = tonic::unary(
        mock_server.address,
        "myservice/getItems",
        AnyRequest {"id": 42},
    )
        .header("locale", "en-US")
        .insecure()

    assert_eq!(matches!(Ok(expected), response))
```

## Test isolation

Each instance of `MockServer` is fully isolated: `start` takes care of finding a random port available on your local machine which is assigned to the new `MockServer`.

To ensure full isolation and no cross-test interference, `MockServer`s shouldn't be shared between tests. Instead, `MockServer`s should be created in the test where they are used.

When a `MockServer` instance goes out of scope (e.g. the test finishes), the corresponding HTTP server running in the background is shut down to free up the port it was using.

## Efficiency

`gRusticMock` maintains a pool of mock servers in the background to minimise the number of connections and the time spent starting up a new `MockServer`.

## References

This repository was inspired by:
- [`wiremock-rs`](https://github.com/LukeMathWalker/wiremock-rs) for HTTP mocking to test Rust applications.
- [`grpcmock`](https://github.com/nhatthm/grpcmock) for gRPC mocking to test Go applications.
