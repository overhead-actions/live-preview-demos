# Live Preview Demo

This is a demo for the Live Preview project:
- Ngrok Version: [github.com/overhead-actions/live-preview-ngrok](https://github.com/overhead-actions/live-preview-ngrok)
- FRP (Fast Reverse Proxy) Version: [github.com/overhead-actions/live-preview-frp](https://github.com/overhead-actions/live-preview-frp)

The Live Preview project aims to provide temporary validation deployments that make the process of reviewing a PR easier, by providing an environment in which the reviewer can verify if the changes are good-to-go from a user perspective. Its main advantage is to remove the requirement for pulling the branch code, building and running it locally.

Whenever the action is triggered, it will pull the application's Docker image (or images, if the project requires a docker-compose with multiple services), run it and magically expose it to the reviewers, posting the link to the application in a Pull Request comment. The magic can be handled either by [Ngrok](https://ngrok.com/), a freemium tunneling service, or an open-source, self-hosted, [FRP (Fast Reverse Proxy)](https://github.com/fatedier/frp) server that works as a reverse proxy and tunneling platform.

## Workflows

#### FRP (Fast Reverse Proxy)

```yml
name: Live Preview FRP

on: pull_request

jobs:
  default:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Start services
        run: docker-compose up -d

      - name: Start tunnel
        uses: overhead-actions/live-preview-frp@main
        with:
          domain: ${{ github.head_ref }}.arthurbdiniz.com

      - name: Comment PR
        uses: unsplash/comment-on-pr@v1.3.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          msg: 'Here is your live preview URL 🚀: http://${{ github.head_ref }}.arthurbdiniz.com'
          check_for_duplicate_msg: false

      - name: Wait
        run: sleep 300
```

#### Ngrok

```yml
name: Live Preview Ngrok

on: pull_request

jobs:
  default:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Start services
        run: docker-compose up -d

      - name: Start tunnel
        uses: overhead-actions/live-preview@main
        with:
          protocol: http
          port: 4000
          ngrok_auth_token: ${{ secrets.NGROK_AUTH_TOKEN }}

      - name: Get URL
        id: vars
        run: echo "::set-output name=url::$(curl -s localhost:4040/api/tunnels | jq -r .tunnels[0].public_url)"

      - name: Comment PR
        uses: unsplash/comment-on-pr@v1.3.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          msg: 'Here is your live preview URL 🚀: ${{ steps.vars.outputs.url }}'
          check_for_duplicate_msg: false

      - name: Wait
        run: sleep 300
```

## Authors

 - Arthur Diniz: <arthurbdiniz@gmail.com>
 - Victor Moura: <victor_cmoura@hotmail.com>

## License

MIT Licensed. See [LICENSE](https://github.com/overhead-actions/live-preview-demos/blob/master/LICENSE) for full details.