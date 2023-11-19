---
title:  "Upgrade rust logs to structured"
layout: post
author: tsionyx
date:   2023-11-19 16:38:30 +0500
categories: rust
---

Make your rust logs structured using `tracing` library.

In an attempt to better understand what's going on in your
concurrent async program you could use logs,
but they will appear interleaved in the resulted output,
so we should use some context with every process,
aka _structured logging_.

For the first attempt we should replace all the occurrences of
the `log::` macros with the same set of `tracing::` macros.

The same goes for the application logic where you set up the log
levels, destinations, filtering, etc.
Just replace the `env_logger` with
(almost the same at first glance, but `tracing` counterpart):
`tracing_subscriber::fmt`.

```shell
# find all files where the word 'log' appears
# and replace its usage with the 'tracing' 
git grep -lw log | xargs sed -i 's/use log::/use tracing::/'

# find all files where the word 'env_logger' appears
# and replace its usage with the 'tracing_subscriber::fmt'
git grep -lw env_logger | xargs sed -i 's/env_logger::/tracing_subscriber::fmt::/'

# adjust formatting, some lines could become out-of-order
cargo fmt --all
```

Also, replace your dependencies in _Cargo.toml_:

```toml
# remove the 'log' dependency
tracing = { version = "*", default_features = false, features = ["std", "log"]}

# remove the 'env_logger'
tracing-subscriber = { version = "*", features = ["env-filter", "parking_lot", "time"] }
```
