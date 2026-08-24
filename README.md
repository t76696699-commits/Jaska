# ============================================================
# 1) Ushbu repo'ning haqiqiy tanlovi - GitHub-hosted, uchala faylda ham
# ============================================================
# test.yml:
jobs:
  backend:
    runs-on: ubuntu-latest    # <- GitHub-hosted
  frontend:
    runs-on: ubuntu-latest    # <- GitHub-hosted

# deploy-backend.yml:
jobs:
  deploy:
    runs-on: ubuntu-latest    # <- GitHub-hosted (SSH orqali PROD serverga ulanadi)

# deploy-frontend.yml:
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest    # <- GitHub-hosted (build shu yerda, keyin rsync)

# ============================================================
# 2) Agar self-hosted ishlatilganda - deploy-backend.yml QANDAY o'zgarardi
# ============================================================
# HOZIRGI (GitHub-hosted + SSH):
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          mkdir -p ~/.ssh && printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
      - name: Pull and restart (orqali SSH)
        run: ssh -i ~/.ssh/deploy_key user@host "cd /app && git pull && systemctl restart app"

# MUQOBIL (self-hosted, prod serverning o'zida ishlaydi):
jobs:
  deploy:
    runs-on: self-hosted       # <- prod serverning o'zi runner sifatida ro'yxatdan o'tgan
    steps:
      # SSH kaliti, ssh-keyscan, Configure SSH step'i - UMUMAN KERAK EMAS,
      # chunki runner ALLAQACHON prod serverning o'zi.
      - run: cd /app && git pull origin server
      - run: source venv/bin/activate && pip install -r requirements.txt --quiet
      - run: sudo systemctl restart student-platform-backend

# ============================================================
# 3) Self-hosted runner'ni repo'ga ro'yxatdan o'tkazish (bir martalik)
# ============================================================
$ mkdir actions-runner && cd actions-runner
$ curl -o actions-runner-linux-x64.tar.gz -L \
    https://github.com/actions/runner/releases/download/v2.319.1/actions-runner-linux-x64-2.319.1.tar.gz
$ tar xzf actions-runner-linux-x64.tar.gz
$ ./config.sh --url https://github.com/OWNER/REPO --token <RUNNER_TOKEN>
$ ./run.sh
# Runner endi Settings -> Actions -> Runners bo'limida "Idle" (bo'sh)
# holatida ko'rinadi va runs-on: self-hosted bo'lgan har qanday job'ni
# kutib turadi.

# ============================================================
# 4) Label orqali muayyan self-hosted runner'ni tanlash
# ============================================================
jobs:
  gpu-training:
    runs-on: [self-hosted, gpu, linux]   # <- faqat shu 3 ta labelga ega
                                          #    runner'da ishga tushadi
    steps:
      - run: nvidia-smi
      - run: python train_model.py

# ============================================================
# 5) Xavfsizlik: public repo'da self-hosted runner ishlatishning xavfi
# ============================================================
# Agar OWNER/REPO PUBLIC bo'lsa va self-hosted runner ulangan bo'lsa:
#   1. Tashqi odam fork qilib, zararli kod bilan PR ochadi
#   2. Agar workflow pull_request_target yoki noto'g'ri sozlangan
#      pull_request trigger bilan ishlasa, bu kod SIZNING self-hosted
#      runner'ingizda BAJARILISHI mumkin
#   3. Runner joylashgan tarmoqqa (masalan ichki baza, boshqa serverlar)
#      kirish xavfi tug'iladi
# Shu sababli GitHub PUBLIC repo'lar uchun self-hosted'ni faqat qattiq
# nazorat (masalan required approval for first-time contributors) bilan
# ishlatishni tavsiya qiladi.
