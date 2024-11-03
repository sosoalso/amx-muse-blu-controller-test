BLU 제품군 컨트롤러

내가 잘 못하는 건지 버그인 진 모르겠는데, blu device 의 component 에서 값을 (.value) 직접 참조하거나 하면
드르륵 3번 읽히는 증상이 보기 싫어서, component 의 state의 값이 변경될 때 감지해서 저장해놓고
그 값을 기준으로 버튼 이벤트와 피드백이 왔다갔다 하도록 함..

```python
# component state 저장
BLU_COMPONENT_STATES = BluComponentState()

# blu 컨트롤러 - blu device와 component state를 인자로 넣기
blu_controller = BluController(DV_BLU, BLU_COMPONENT_STATES)

# blu device 가 온라인되면 BluController 인스턴스에 내가 제어하기 원하는
# component 의 path 를 [("module name", ... "channel")]..[] 리스트 들로 넣으면 됨
# 

BLU_PATH_MAIN_MIXER_GAIN = [("Main Mixer", f"Channel {index_ch}", "Gain") for index_ch in range(1, 12 + 1)] + [
    ("Main Mixer", "Main L", "Gain"),
    ("Main Mixer", "Main R", "Gain"),
    ("Main Mixer", f"Aux {1}", "Gain"),
    ("Main Mixer", f"Aux {2}", "Gain"),
]
BLU_PATH_MAIN_MIXER_MUTE = [("Main Mixer", f"Channel {index_ch}", "Mute") for index_ch in range(1, 12 + 1)] + [
    ("Main Mixer", "Main L", "Gain"),
    ("Main Mixer", "Main R", "Mute"),
    ("Main Mixer", f"Aux {1}", "Gain"),
    ("Main Mixer", f"Aux {2}", "Mute"),
]

def handle_blu_controller_online(*args):
    blu_controller.init(BLU_PATH_MAIN_MIXER_MUTE, BLU_PATH_MAIN_MIXER_GAIN)

blu_controller.device.online(handle_blu_controller_online)


# component path 를 tuple 로 받음
def ui_refresh_blu_button_by_path(path):
    if path in BLU_PATH_MAIN_MIXER_MUTE:
        idx = BLU_PATH_MAIN_MIXER_MUTE.index(path)
        ch_index = 11 + idx
        val = blu_controller.component_states.get_state(path)  # 저장된 component_states 에서 해당 path 값 가져오기
        if val is not None:
            for tp in TP_LIST:
                tp_set_button(tp, 4, ch_index, val == "Muted") # Mixer 나 Gain 같은 애들은 값이 "Muted" 인지 확인해서 tp 버튼 피드백..

# 각 터치판넬에 버튼 이벤트 추가하기
def tp_add_blu_button_event(evt):
   tp = evt.source // 터치판넬 디바이스
   for idx, path in enumerate(BLU_PATH_MAIN_MIXER_MUTE):  # 리스트에서 가져온 의 component path 와 idx 
        btn_index = 11 + idx
        mute_toggle_btn = ButtonHandler()
        mute_toggle_btn.add_event_handler("push", lambda path=path: blu_controller.toggle_muted_unmuted(path)) # 눌렀을 때 뮤트 토글
        tp_add_watcher(tp, 4, btn_index, mute_toggle_btn.handle_event) # 버튼 이벤트 등록
        ui_refresh_blu_button_by_path(path)  # 이건 그냥 온라인 됐을 때 한 번 버튼 업데이트 되라고 넣음

for tp in TP_LIST:
    tp.online(tp_add_blu_button_event)
