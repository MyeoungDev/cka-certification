# Lecture 203 - Volume Driver Plugins in Docker

## Docker의 Volume Driver Plugin

- Volume 은 Storage Driver 가 아닌 Volume Driver Plugin 을 통해 관리된다.
- 기본적으로 Docker 는 Local Volume Driver Plugin 을 사용한다.
- Local Volume Driver Plugin 은 Docker 호스트에 볼륨을 생성하고 `/var/lib/docker/volumes` 디렉토리에 데이터를 저장한다.
- 볼륨 드라이버 플러그인을 선택할 때는 스토리지 인프라 요구 사항을 고려해야 한다.
- `Azure File Storage`, `Google Compute Persistent Disks` 와 같은 타사 스토리지 솔류션에 볼륨을 생성할 수 있는 다른 많은 볼륨 드라이버 플러그인도 사용 가능하다.
- 특정 볼륨 드라이버 플러그인을 사용하여 클라우드에 프로비저닝하여 컨테이너 데이터를 클라우드에 안전하게 유지할 수 있다.

```bash
docker run -it \\
  --name mysql \\
  --volume-driver rexray/ebs \\
  --mount src=ebs-vol,target=/var/lib/mysql \\
  mysql

```