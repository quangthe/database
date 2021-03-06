# Contributing

## Development

_Requirements: [node.js][node-js] (including npm)_

Before you start developing, you should clone or download this repository and run:

```bash
$ npm install
```

As described in the [spec][spec] the build will automatically update the `dist` branch with the generated data when a pull request is merged into `master`. But, to check your changes you can generate `build/out/vX/diacritics.json` yourself by running:

```bash
$ npm run build
```

## Contribution and License Agreement

If you contribute to this project, you are implicitly allowing your code to be distributed under [this license][license]. You are also implicitly verifying that all code is your original work.

[node-js]: https://nodejs.org/en/
[spec]: ./spec/
[license]: https://git.io/vXg2H
