# Open WebUI Troubleshooting Guide

## Lỗi Node.js Version Conflict

1. Xác minh cài đặt nvm và Node.js:
# Kiểm tra danh sách phiên bản đã cài
nvm list
# → Phải thấy v18.13.0 trong danh sách

2. Set phiên bản mặc định:
nvm alias default 18.13.0
exec zsh # Load lại shell

3. Trong thư mục project:
cd ~/test/open-webui
nvm use 18.13.0

# Xóa cache cũ
rm -rf node_modules package-lock.json

# Cài đặt lại với npm 8
npm install -g npm@8.19.4
npm install --legacy-peer-deps

4. Verify cuối cùng:
node -v # v18.13.0
npm -v  # 8.19.4

So sánh các phiên bản Node.js:
| Phiên bản | Trạng thái | Hỗ trợ đến | Phù hợp với Open WebUI |
|-----------|---------------|-----------------|-----------------------|
| 18.17.1 | Active LTS | April 2025 | ✅ Tốt nhất |
| 20.11.1 | Active LTS | April 2026 | ✅ Tốt |
| 22.0.0 | Current | October 2024 | ⚠️ Cần test thêm |


PHẦN BACKEND
1. Cài Miniconda cho macOS:
# Tải script cài đặt
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh

# Chạy installer
bash Miniconda3-latest-MacOSX-arm64.sh

# Khởi động lại terminal
exec zsh

2. Tạo Conda environment cho Open WebUI:
cd ~/test/open-webui/backend
conda create -n open-webui python=3.11
conda activate open-webui

3. Cài dependencies backend:
pip install -r requirements.txt -U

4. Chạy backend server:
cd ~/test/open-webui/backend
conda activate open-webui (nếu chưa active)
sh dev.sh
Phải kiểm 
which python
# Phải hiển thị đường dẫn Conda environment:
# /Users/yourname/miniconda3/envs/open-webui/bin/python
vì nếu sai / quên backend sẽ dùng Python hệ thống → lỗi dependencies
(Đã tạo alias ow-backend như sau:
echo 'alias ow-backend="conda activate open-webui && cd ~/test/open-webui/backend && sh dev.sh"' >> ~/.zshrc
source ~/.zshrc)

PHẦN FRONTEND
1. Build frontend production:
cd ~/test/open-webui
npm run build

2. Kiểm tra thư mục build:
ls build/
# Phải thấy các file index.html, assets/

NẾU KHÔNG BUILD ĐƯỢC
vd bị lỗi thấy do SvelteKit
do check 
npm list @sveltejs/kit
# Phải là ^2.5.20

thì Đồng bộ phiên bản SvelteKit:
cd ~/test/open-webui
npm uninstall @sveltejs/kit
npm install @sveltejs/kit@2.5.20 --save-exact
npm list @sveltejs/kit
# Kết quả phải là: @sveltejs/kit@2.5.20

Cập nhật các package liên quan:
npm install @sveltejs/adapter-static@3.0.2 --save-exact

Xóa cache và build lại:
rm -rf node_modules package-lock.json .svelte-kit build
npm install

Chạy lại build với log chi tiết: từ đây copy log cho AI debug
npm run build -- --debug 2>&1 | grep -i 'error\|warning'

Phát hiện ra lỗi trong doc có đề cập "FATAL ERROR: Reached Heap Limit"
https://docs.openwebui.com/getting-started/advanced-topics/development

1. Tăng bộ nhớ cho Node.js:
export NODE_OPTIONS="--max-old-space-size=8192"

2. Build lại với thêm RAM:
npm run build -- --emptyOutDir

3. Nếu dùng macOS ARM (M1/M2), thêm cờ:
NODE_OPTIONS="--max-old-space-size=8192" npm run build -- --emptyOutDir

Cấu hình permanent:
Thêm vào ~/.zshrc:
export NODE_OPTIONS="--max-old-space-size=8192"

