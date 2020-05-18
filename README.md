# meilisearch-sdk [![Latest Version]][crates.io]
[Latest Version]: https://img.shields.io/crates/v/meilisearch-sdk
[crates.io]: https://crates.io/crates/meilisearch-sdk

MeiliSearch Rust is a client for [MeiliSearch](https://www.meilisearch.com/) written in Rust.
[MeiliSearch](https://www.meilisearch.com/) is a powerful, fast, open-source, easy to use and deploy search engine.
Both searching and indexing are highly customizable.
Features such as typo-tolerance, filters, and synonyms are provided out-of-the-box.

### Table of Contents <!-- omit in toc -->
- [🔧 Installation](#-installation)
- [🚀 Getting started](#-getting-started)
- [🌐 Running in the browser with WASM](#-running-in-the-browser-with-wasm)
- [🤖 Compatibility with MeiliSearch](#-compatibility-with-meilisearch)

## 🔧 Installation

This crate requires a MeiliSearch server to run. See [here](https://docs.meilisearch.com/guides/advanced_guides/installation.html#download-and-launch) to install and run MeiliSearch.

Then, put `meilisearch-sdk = "0.1"` in your Cargo.toml, as usual.

Using this crate is possible without [serde](https://crates.io/crates/serde), but a lot of features require serde.
Add `serde = {version="1.0", features=["derive"]}` in your Cargo.toml.

## 🚀 Getting Started

Here is a quickstart for a search request (please follow the [installation](#-installation) steps before)

```rust
use meilisearch_sdk::{document::*, indexes::*, client::*, search::*};
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Book {
    book_id: usize,
    title: String,
}

// That trait is required to make a struct usable by an index
impl Document for Book {
    type UIDType = usize;

    fn get_uid(&self) -> &Self::UIDType {
        &self.book_id
    }
}

// Create a client (without sending any request so that can't fail)
let client = Client::new("http://localhost:7700", "");

// Get the index called "books"
let mut books = client.get_or_create("books").unwrap();

// Add some books in the index
books.add_documents(vec![
    Book{book_id: 123,  title: String::from("Pride and Prejudice")},
    Book{book_id: 456,  title: String::from("Le Petit Prince")},
    Book{book_id: 1,    title: String::from("Alice In Wonderland")},
    Book{book_id: 1344, title: String::from("The Hobbit")},
    Book{book_id: 4,    title: String::from("Harry Potter and the Half-Blood Prince")},
    Book{book_id: 42,   title: String::from("The Hitchhiker's Guide to the Galaxy")},
], Some("book_id")).unwrap();

// Query books (note that there is a typo)
let query = Query::new("harry pottre");
println!("{:?}", books.search::<Book>(&query).unwrap().hits);
```

Output:

```rust
[Book { book_id: 4, title: "Harry Potter and the Half-Blood Prince" }]
```

## 🌐 Running in the browser with WASM

This crate fully supports WASM. However, there are some syntax differences between a native and a WASM program using `meilisearch-sdk`.
That means that you can't use the exact same code for native and web programs but it is very similar.
Only some `.await` are to be added on a native program to make a working Wasm program. (Because all `meilisearch-sdk`'s methods are `async` on Wasm and `sync` on native target (but `async` is planned for native too))
However, making a program intended to run in a web browser requires a **very** different design than a CLI program. To see an example of a simple Rust web app using meilisearch, see [tutorial todo here]().

WARNING: `meilisearch-sdk` will panic if no Window is available (ex: Web extension).

## 🤖 Compatibility with MeiliSearch

This crate is currently supporting MeiliSearch v10.0 and will be maintained.

## Running the tests

All the tests are documentation tests.
Since they are all making operations on the MeiliSearch server, running all the tests simultaneously would cause panics.
To run the tests one by one, run `cargo test -- --test-threads=1`.

Current version: 0.1.3

License: MIT
