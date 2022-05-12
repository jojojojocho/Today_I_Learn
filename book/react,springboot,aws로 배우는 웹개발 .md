### 대략적인 아키텍처

#### HTML/CSS/REACT.JS(JS프레임워크) : 프론트엔드 APP 개발에 사용. / 프론트 서버 -> React.js app 반환
#### SPRING BOOT : 백엔드 APP 개발에 사용. / 프론트엔드 APP이 사용할 REST API 구현/ 백엔드 서버 -> REST API 반환. / REST API로 인한 MSA로의 확장이 용이.
#### AWS : 프론트엔드와 백엔드 APP이 실행될 프로덕션 환경 구축.
(VPC (Virtual Private Cloud): AWS의 ROUTE53 / 이 안에 프론트 서버, 백엔드 서버가 들어간다.)

