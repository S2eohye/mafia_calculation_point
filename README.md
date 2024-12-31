import streamlit as st
import pandas as pd
from pathlib import Path

# 데이터 저장 경로 설정
DATA_DIR = Path("data")
DATA_FILE = DATA_DIR / "user_data.csv"

# 사용자 이름과 비밀번호를 매핑한 딕셔너리
USER_CREDENTIALS = {
    "user1": "pass123",
    "user2": "secure456",
    "admin": "admin789"
}

# 데이터 디렉토리 확인
DATA_DIR.mkdir(exist_ok=True)

# 로그인 상태 관리
if "authenticated" not in st.session_state:
    st.session_state.authenticated = False
if "current_user" not in st.session_state:
    st.session_state.current_user = None

# 로그인 페이지
def login_page():
    st.title("로그인 페이지")

    name = st.text_input("이름")
    password = st.text_input("비밀번호", type="password")

    if st.button("로그인"):
        if name in USER_CREDENTIALS and USER_CREDENTIALS[name] == password:
            st.session_state.authenticated = True
            st.session_state.current_user = name
            st.experimental_rerun()  # 로그인 후 페이지 새로고침
        else:
            st.error("이름 또는 비밀번호가 잘못되었습니다.")

# 사용자 입력 페이지
def user_input_page():
    st.title("사용자 입력 페이지")
    st.write(f"환영합니다, {st.session_state.current_user}님!")

    # 사용자 입력값
    name = st.text_input("이름을 입력하세요:")
    age = st.number_input("나이를 입력하세요:", min_value=0, max_value=120, step=1)
    comments = st.text_area("하고 싶은 말을 입력하세요:")

    # 입력값 저장
    if st.button("제출"):
        if name and age:
            # 데이터 프레임 생성
            new_data = pd.DataFrame({
                "이름": [name],
                "나이": [age],
                "메시지": [comments],
                "작성자": [st.session_state.current_user]
            })
            
            # 데이터 저장
            if DATA_FILE.exists():
                existing_data = pd.read_csv(DATA_FILE)
                updated_data = pd.concat([existing_data, new_data], ignore_index=True)
            else:
                updated_data = new_data
            
            updated_data.to_csv(DATA_FILE, index=False)
            st.success("입력값이 저장되었습니다!")
        else:
            st.error("모든 필드를 채워주세요.")

    if st.button("로그아웃"):
        st.session_state.authenticated = False
        st.session_state.current_user = None
        st.experimental_rerun()

# 로그인 상태에 따라 적절한 페이지 표시
if not st.session_state.authenticated:
    login_page()
else:
    user_input_page()
