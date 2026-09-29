# 리소스 정리 체크리스트

정리 일자: 2026-09-29 / 리전: 서울(ap-northeast-2) / 계정: IAM 사용자

## 정리 대상과 확인 결과

| 완료 | 리소스 | 확인 위치 | 근거 |
| --- | --- | --- | --- |
| [x] | EC2 `b3-1-web` — 종료됨(Terminated) | EC2 → 인스턴스 | `step_8_01_ec2_terminated.png` |
| [x] | EBS 볼륨(8GiB, 미사용 포함) — 목록 비어 있음 | EC2 → 볼륨 | `step_8_02_ebs_empty.png` |
| [x] | Elastic IP — 할당한 적 없음(퍼블릭 IP는 서브넷 자동 할당), 목록 비어 있음 | EC2 → 탄력적 IP | `step_8_03_eip_empty.png` |
| [x] | Security Group `b3-1-web-sg` — 삭제 | EC2 → 보안 그룹 | (VPC 삭제 성공으로 갈음) |
| [x] | Subnet `10.0.1.0/24`, Route Table — 삭제 | VPC → 서브넷·라우팅 테이블 | (VPC 삭제 성공으로 갈음) |
| [x] | Internet Gateway — VPC에서 분리 후 삭제 | VPC → 인터넷 게이트웨이 | `step_8_04_igw_deleted.png` |
| [x] | VPC `10.0.0.0/16` — 삭제 | VPC → VPC | `step_8_05_vpc_deleted.png` |
| — | NAT Gateway · ELB/ALB · RDS | — | 생성하지 않아 해당 없음 |
| — | 키페어 · IAM 사용자 | — | 과금 없음, 유지 |

## 삭제 순서와 이유

1. **EC2 종료 → EBS 확인**: 인스턴스가 살아 있으면 시간당 과금이 계속되고, 인스턴스가 네트워크 인터페이스를 잡고 있어 SG·서브넷도 지워지지 않는다. 루트 볼륨은 종료 시 같이 삭제되는 게 기본값이지만, 분리돼 남은 볼륨은 종료 후에도 저장 용량만큼 과금되므로 목록으로 확인한다.
2. **Elastic IP 확인**: 인스턴스에 붙지 않은 EIP는 그 자체로 과금된다. 할당한 적이 없어도 목록으로 확인한다.
3. **SG → Subnet → Route Table**: 안쪽 리소스부터 지운다. VPC 안에 리소스가 남아 있으면 VPC가 삭제되지 않는다.
4. **IGW 분리 → 삭제 → VPC 삭제**: IGW는 VPC에 붙어 있는 동안 삭제할 수 없어서 분리를 먼저 한다.

※ `step_8_04`·`step_8_05`에 남아 있는 IGW·VPC(`172.31.0.0/16`)는 계정 기본(default) VPC의 것으로 실습 리소스가 아니다. 실습 VPC `10.0.0.0/16`과 그 IGW는 목록에 없다.

## 과금 확인
- Billing Dashboard는 IAM 사용자에게 접근 권한을 열지 않아 확인하지 못했다(루트 계정 사용 금지 제약). 대신 위 리소스 목록으로 잔존 리소스가 없음을 확인했다.