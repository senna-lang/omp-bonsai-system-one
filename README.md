# omp-bonsai-system-one

A plugin for the [omp](https://github.com/can1357/oh-my-pi) coding agent that runs omp's `judge` role on a local System One model: [bonsai-4b-system-one](https://github.com/senna-lang/bonsai-4b-system-one), a frozen Ternary-Bonsai-4B served over HTTP. The judge drives omp's per-prompt `auto` thinking level and its other typed judgments; with this plugin they run offline and at no cost.

An optional `router` feature also switches the session's model once, from the first prompt's rated difficulty, to your own roles:

| Rated difficulty | Role |
| --- | --- |
| light: a question about the code, or a tiny mechanical edit | `@smol` |
| standard: an ordinary feature, fix, test, or review within one area | `@mid`, or `@default` when no `mid` role is set |
| complex: subtle design, cross-cutting change, or hard debugging | `@slow` |

The plugin assigns no models; a tier whose roles are unset keeps the current model.

> Experimental. Not affiliated with TypeSafe, PrismML, or the omp project.

## Install

1. Install and run the local server from [bonsai-4b-system-one](https://github.com/senna-lang/bonsai-4b-system-one#local-server-system-one-api-format) (Apple Silicon: MLX, about 1.1 GB of weights).
2. Install the plugin from npm:

   ```bash
   omp plugin install 'omp-bonsai-system-one[router]'     # drop [router] for the judge only
   ```

   or from this repository as a marketplace:

   ```bash
   omp plugin marketplace add senna-lang/omp-bonsai-system-one
   omp plugin install omp-bonsai-system-one@omp-bonsai-system-one
   omp plugin features omp-bonsai-system-one --enable router     # optional
   ```

3. Select the model for the judge role in `~/.omp/agent/config.yml`:

   ```yaml
   modelRoles:
     judge: bonsai-local/bonsai-4b-system-one
   defaultThinkingLevel: auto
   ```

## Starting the server

Keep `python -m system_one_bonsai.serve` running yourself, or let the plugin start it on the first prompt of a session by pointing it at your environment:

```bash
export BONSAI_SYSTEM_ONE_SERVE_CMD="/path/to/bonsai-4b-system-one/.venv/bin/python -m system_one_bonsai.serve"
```

The first prompt waits up to 20 s for the server to come up. The started server keeps running after omp exits (about 1.5 GB resident on MLX); its log is `~/.cache/bonsai-system-one/serve.log`. `BONSAI_SYSTEM_ONE_URL` overrides the default address `http://127.0.0.1:8765`.

If the server is unavailable, the turn continues: auto thinking keeps its previous level and the router keeps the current model.

## Accuracy

On 80 coding-agent requests with reference effort levels from a strong model (Claude Opus; not human labels), asked omp's own auto-thinking question through omp's judge path:

| Judge | Exact effort | Within one level | Median latency (Apple M2) |
| --- | ---: | ---: | ---: |
| `bonsai-local/bonsai-4b-system-one` | **0.66** | **0.99** | 0.8 s |
| omp built-in `local/lfm2-1.2b` | 0.29 | 0.69 | 3.0 s |
| omp built-in `local/lfm2.5-230m` | 0.25 | 0.56 | 0.3 s |

It tends to rate one level low, mostly `high` for `xhigh` requests. The set is small and the reference labels come from a single model.

## License

Apache-2.0. See [`LICENSE`](LICENSE).
