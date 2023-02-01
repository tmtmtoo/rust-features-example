# rust-features-example

`--features` フラグを指定した場合の依存関係を調査した。

## tl;dr

- `Cargo.toml` の `[features]` に `default` を指定していると、`--features` の入力にかかわらず、該当の依存クレートを含めてビルドが実行される。
- `default` を指定しない場合は、各 `features` に応じた依存クレートのみがビルド対象となる。

## 前提となる `Cargo.toml`

```toml
[features]
default = [ "foo" ]
foo = [ "derive-new" ]
bar = [ "boolinator" ]

[dependencies]
derive-new = { version = "0.5.9", default_features = false, optional = true }
boolinator = { version = "2.4.0", default_features = false, optional = true }
```

`cargo tree` による依存クレートの表示

```
❯ cargo tree --features foo
rust-features-example v0.1.0 (/***/rust-features-example)
└── derive-new v0.5.9 (proc-macro)
    ├── proc-macro2 v1.0.50
    │   └── unicode-ident v1.0.6
    ├── quote v1.0.23
    │   └── proc-macro2 v1.0.50 (*)
    └── syn v1.0.107
        ├── proc-macro2 v1.0.50 (*)
        ├── quote v1.0.23 (*)
        └── unicode-ident v1.0.6
```

```
❯ cargo tree --features bar
rust-features-example v0.1.0 (/***/rust-features-example)
├── boolinator v2.4.0
└── derive-new v0.5.9 (proc-macro)
    ├── proc-macro2 v1.0.50
    │   └── unicode-ident v1.0.6
    ├── quote v1.0.23
    │   └── proc-macro2 v1.0.50 (*)
    └── syn v1.0.107
        ├── proc-macro2 v1.0.50 (*)
        ├── quote v1.0.23 (*)
        └── unicode-ident v1.0.6
```

_`default` で `foo` を指定している場合、`bar` にも、`foo` の依存クレートも依存関係となる。_

## `default` を指定しない `Cargo.toml`

```toml
[features]
foo = [ "derive-new" ]
bar = [ "boolinator" ]

[dependencies]
derive-new = { version = "0.5.9", default_features = false, optional = true }
boolinator = { version = "2.4.0", default_features = false, optional = true }
```

`cargo tree` による依存クレートの表示

```
❯ cargo tree --features foo
rust-features-example v0.1.0 (/***/rust-features-example)
└── derive-new v0.5.9 (proc-macro)
    ├── proc-macro2 v1.0.50
    │   └── unicode-ident v1.0.6
    ├── quote v1.0.23
    │   └── proc-macro2 v1.0.50 (*)
    └── syn v1.0.107
        ├── proc-macro2 v1.0.50 (*)
        ├── quote v1.0.23 (*)
        └── unicode-ident v1.0.6
```

```
❯ cargo tree --features bar
rust-features-example v0.1.0 (/***/rust-features-example)
└── boolinator v2.4.0
```

_`bar` では `foo` で指定しているクレートに依存していない。_
