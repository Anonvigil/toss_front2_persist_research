# Toss Front2 `persist` 공개용 이미지

[`Front2_persist_sanitized.ext4`](https://github.com/Anonvigil/toss_front2_persist_research/releases/download/persist-public-v1/Front2_persist_sanitized.ext4)는 원본 `persist.bin`에서 기기 식별자와 실행 상태의 일부를 마스킹하고, Gatekeeper 보안 저장소 묶음을 제거해 **새로 만든 ext4 이미지**입니다. 원본 백업은 수정하지 않았습니다. 이미지 크기는 원본과 같은 33,554,432바이트(32 MiB)이고, 공개본에는 파일 229개와 디렉터리 33개가 있습니다.

마스킹하지도 제거하지도 않은 파일의 **내용은 원본과 바이트 단위로 같습니다.** 남겨 둔 파일의 크기와 파일 내부 오프셋도 같습니다. 새 ext4를 만들었으므로 물리 블록 배치는 원본과 다를 수 있습니다. 원본의 삭제 파일 흔적, 빈 블록 내용, 저널은 복사하지 않았습니다.

## 마스킹한 위치

| 파일 | 값이 들어가는 위치와 형식 | 처리 |
| --- | --- | --- |
| `/wlan_mac.bin` | `Intf0MacAddress=` 뒤의 16진수 12자리: Wi-Fi MAC 주소 | 12자만 `X`로 변경 |
| `/quec_bt_mac` | `XX:XX:XX:XX:XX:XX` 형식의 Bluetooth MAC 주소 | 숫자·문자 12자만 `X`로 변경하고 `:`는 유지 |
| `/time/ats_1`, `/time/ats_2`, `/time/ats_16` | 각 파일의 8바이트 시간 서비스 상태 | 각 파일의 내용 8바이트를 0으로 변경 |
| `/Prev_SinglePreview_Cam0_Frame40.yuv` | 촬영 프레임 1,843,200바이트 | 파일 내용 전체를 0으로 변경 |

정확한 파일 내부 오프셋과 **원본 `persist.bin` 기준 물리 바이트 오프셋**은 [`mask_map.csv`](mask_map.csv)에 있습니다. 원래 값은 표에 넣지 않았습니다.

## 제거한 위치

`/data/<44자 인코딩 이름>/` 아래의 Gatekeeper 보안 저장소 묶음 전체를 공개본에서 제외했습니다. 원본에는 `vTable`, 20자 인코딩 이름 파일, 44자 인코딩 이름 파일 및 각각의 `.bak`가 있었습니다. 16 KiB 파일 내부의 `gk_sfs_file` 표식과 `keymaster` 펌웨어의 Gatekeeper 등록·검증 코드가 이 묶음을 보안 저장소 상태와 연결합니다. 파일의 암호화 방식이나 내부 인증 이력까지 확인했다는 뜻은 아닙니다.

[`removal_map.csv`](removal_map.csv)는 **원본 이미지에서 제거된 6개 파일의 내용 영역과 7개 디렉터리 엔트리 이름 위치**를 원본 물리 오프셋으로 기록합니다. 인코딩된 실제 이름과 파일 내용은 싣지 않았습니다. `file_offset_hex`가 비어 있는 행은 디렉터리 엔트리의 이름 위치입니다.

두 CSV의 `original_ext4_offset_hex`는 원본 `persist.bin`의 시작 기준입니다. **새 이미지의 물리 오프셋이 아닙니다.** 남아 있는 파일의 마스킹 위치는 `mask_map.csv`의 `file_offset_hex`로도 찾을 수 있습니다.

## 인증 범위

제거한 묶음은 [Android Gatekeeper](https://source.android.com/docs/security/features/authentication/gatekeeper)의 화면잠금·Keystore 인증 상태와 관련될 수 있습니다. 백업된 `persist` 파일에서는 Widevine L1 키박스나 DRM 자격증명을 식별하지 못했습니다. [Google Widevine 설명](https://developers.google.com/widevine/drm/overview)의 provisioning·keybox·OEMCrypto 요소가 다른 저장소에 있는지, 기기의 L1이 활성화되어 있는지는 이 이미지로 판단할 수 없습니다.

## 검증 및 사용 범위

- `e2fsck -fn` 통과. 파일 229개와 디렉터리 33개의 목록을 확인했습니다.
- 마스킹한 파일 6개는 [`mask_map.csv`](mask_map.csv)의 위치 밖에서 원본과 같습니다. 다른 모든 남은 파일은 원본과 바이트 단위로 같습니다.
- 새 이미지의 빈 ext4 블록 6,347개는 모두 0입니다. 원래의 Wi-Fi·Bluetooth MAC, 제거한 보안 저장소의 인코딩 이름과 `gk_sfs_file` 표식은 이미지 전체에서 검출되지 않았습니다.
- 이미지 SHA-256은 [`SHA256SUMS.txt`](SHA256SUMS.txt)에 있습니다.

이 이미지는 **분석·공개용**입니다. MAC이 마스킹되고 인증 상태 파일이 제거되어 기기에 기록하거나 부팅하는 용도로 사용할 수 없습니다. 같은 모델의 다른 기기 백업이 없으므로, 남겨 둔 모든 자료가 기기 간 동일하다고 실증한 것은 아닙니다.

