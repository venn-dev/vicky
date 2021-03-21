# GitHub Actions

It's weird. Some kind of abstraction like Docker but not Docker.

This is an example configuration yaml for Django:

```yml
name: Django CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.8
      uses: actions/setup-python@v1
      with:
        python-version: 3.8
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run Tests
      run: |
        python manage.py test
      env:
        SECRET_KEY: "thisisthesecretkey"
        DATABASE_URL: "postgres://postgres:postgres@localhost:5432/postgres"

  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/master'
    needs: build
    steps:
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh/
          echo "$SSH_KEY" > ~/.ssh/heartfort.key
          chmod 600 ~/.ssh/heartfort.key
          cat >> ~/.ssh/config << EOF
          Host heartfort
            HostName heartfort.com
            User root
            IdentityFile ~/.ssh/heartfort.key
            StrictHostKeyChecking no
          EOF
          echo DATABASE_URL=$DATABASE_URL > ~/sshenv
          scp ~/sshenv heartfort:~/.ssh/environment
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Clone repository
        run: |
          ssh heartfort "rm -rf /opt/apps/srdce"
          ssh heartfort 'cd /opt/apps/ && git clone git@github.com:sirodoht/srdce.git --config core.sshCommand="ssh -i ~/.ssh/id_rsa_github_deploy_key"'
      - name: Install requirements
        run: ssh heartfort 'cd /opt/apps/srdce && python3 -m venv venv && . venv/bin/activate && pip3 install -r requirements.txt'

      - name: Collect static
        run: ssh heartfort "cd /opt/apps/srdce && . venv/bin/activate && DATABASE_URL=$DATABASE_URL python3 manage.py collectstatic --noinput"
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Run migrations
        run: ssh heartfort "cd /opt/apps/srdce && . venv/bin/activate && DATABASE_URL=$DATABASE_URL python3 manage.py migrate"
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Reload server
        run: ssh heartfort 'touch /etc/uwsgi/vassals/srdce.ini'
```

You have jobs, and each job has steps. It can also depend on other jobs.

A step can be inheritable. Like Docker's `FROM`. E.g. `uses: actions/checkout@v2`. The `uses` key means
that it will draw this code to execute: [actions/checkout](https://github.com/actions/checkout).

If your step has `run` then it just runs that in the shell.

For using environment variable, one can add them in the repository settings and if they are a secret,
to use them like `DATABASE_URL: ${{ secrets.DATABASE_URL }}`.

To run a job only on master, we use `if: github.ref == 'refs/heads/master'`.

If you're doing a static website, then it can be like this:

```
name: Deploy mdBook

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/master'
    steps:
      - uses: actions/checkout@v2

      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh/
          echo "$SSH_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          cat >> ~/.ssh/config << EOF
          Host venn
            HostName wiki.venn.dev
            User root
            IdentityFile ~/.ssh/id_rsa
            StrictHostKeyChecking no
          EOF
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}

      - name: Install mdbook
        run: |
          wget https://github.com/rust-lang/mdBook/releases/download/v0.4.7/mdbook-v0.4.7-x86_64-unknown-linux-gnu.tar.gz
          tar -xzvf mdbook-v0.4.7-x86_64-unknown-linux-gnu.tar.gz
      - name: Build site
        run: ./mdbook build

      - name: Cleanup old files
        run: |
          ssh venn "rm -r /var/www/wiki.venn.dev/"
          ssh venn "mkdir -p /var/www/wiki.venn.dev/html"
      - name: Install requirements
        run: rsync -rsP ./book/ root@wiki.venn.dev:/var/www/wiki.venn.dev/html

      - name: Reload nginx
        run: ssh venn "systemctl reload nginx"
```

Since it's ubuntu (`runs-on: ubuntu-latest`) we can install stuff and use existing tools like `wget` and `tar`.
