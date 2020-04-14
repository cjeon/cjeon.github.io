---
layout: post
title: 커스텀 도메인으로 이메일 수/발신하기
---
<nav>
  <h4>Table of Contents</h4>
  * this unordered seed list will be replaced by toc as unordered list
  {:toc}
</nav>

# 개괄
최근 학부를 졸업하면서 새로운 메일 주소가 필요해졌는데, Gmail 에서 아무리 짱구를 굴려봐도 제가 원하는 주소들은 다 주인이 있더라구요. 그래서 그냥 제가 가지고 있는 도메인으로 이메일 주소를 만들기로 했습니다.  

우여곡절을 좀 겪었고 삽질도 좀 했는데, 인터넷에서 자료를 찾기 힘들었어서 정리해서 기록해봅니다.

우선, 아래 내용을 다룰 거고요,

1. 가지고 있는 도메인으로 이메일 수신하기
2. 가지고 있는 도메인으로 이메일 발신하기

아래는 다루지 않을 것입니다.

1. 도메인을 구입하는 방법
2. 도메인에 대한 기초적인 지식
3. 이메일에 대한 지식
4. AWS SES에 대한 지식

따라서 도메인을 구입해봤거나, 구입할 정도의 지식은 있는 분이 이 글을 읽기 적당할 것 같습니다. 또, AWS 계정도 가지고 계셔야 할 것 같습니다. 도메인 구매 자체는 어렵지 않기 때문에, 원하신다면 먼저 관련 글을 찾아보시고 이 글을 읽어도 될 것 같습니다.

이 글을 그대로 따라하시면 최종 형태는 아래와 같은 형태가 됩니다.

1. 만든 이메일 주소(A)로 이메일을 수신하면 원래 있는 이메일 주소(B)로 이메일이 수신된다.
2. 만든 이메일 주소(A)로 이메일을 발신하면 이메일 주소 A로 발신이 되고, 그 기록은 원래 있는 이메일 주소(B)에 남는다.

B가 필요한 이유는, 수신용 메일 서비스를 만들지 않고, 기존에 존재하는 메일 서비스를 가져다 붙일 것이기 때문입니다. 저는 B에 gmail을 사용했습니다.

A가 new@mydomain.com이고, B가 old@gmail.com이면, 누군가 new@mydomain.com으로 이메일을 보내면 이 이메일은 old@gmail.com에 수신됩니다. 제가 누군가에게 발신인을 new@mydomain.com으로 이메일을 보내면, 받는 사람에게 보낸 사람은 new@mydomain.com 으로 찍히고, 그 보낸 기록은 old@gmail.com에 저장됩니다. 존재하는 것은 주소(new@mydomain.com), 송신용 서비스 (Amazon SES), 수신용 서비스 (gmail) 입니다.

이 글대로 하면 아래와 같은 비용이 듭니다.

1. 도메인 유지비
2. Amazon SES 이메일 송신비

수신만 하는 것에는 비용이 따로 들지 않고, 송신은 보내는 만큼 비용이 청구됩니다. 아직 저도 청구서를 받아보지 못해서 얼마나 나왔다 하는 것을 지금 말씀드리기는 어렵고, AWS의 [공식 문서](https://aws.amazon.com/ses/pricing/)를 참고하세요. 

저는 아래와 같은 셋업에서 글을 설명할 예정이고, 다른 서비스를 쓰는 분은 적절하게 맞춰서 해주시면 될 것 같습니다.

1. Google domains 사용
2. AWS SES 사용
3. Gmail 사용

저는 셋업에 30분 내외, SES의 승인을 받는 데 24시간내외가 소요되었습니다. 

들어가기 전에, 수신, 발신을 모두 해결해주면서 이 문서를 따라가는 것보다 훨씬 간단하고 편한 솔루션이 있습니다. 이 문서보다 비용이 많이 드는 게 단점입니다. Google Gsuite를 쓰시면 아주 간단하게 custom domain으로 이메일 서비스를 만드실 수 있습니다.

또, 저도 삽질하면서 배운 만큼만 알기 때문에 이 글이 완전하고 충분한 지식을 제공한다고 보증하지 않습니다.

서론이 길었습니다. 이제 수신에 대해 이야기해보겠습니다.

# 수신

이번에는 제가 기존에 보유하고 있던 charo.dev 라는 도메인을 이용해서, me@charo.dev로 이메일을 수신하면 기존에 있던 gmail 주소인 주소@gmail.com 으로 수신되게 해보겠습니다.

수신은 아주 간단합니다. MX record 설정으로 해결되는데, 제가 쓰는 domain registrar 에서는 아예 email forwarding 기능이 있어서, mx record를 설정해줄 필요도 없습니다. 많은 registrar이 email forwarding 기능을 제공하는 것으로 알고있기도 하고, 제가 Google domains에서만 작업을 해봤기 때문에, 추가 설명 없이 Google domains 기준으로 설명드리겠습니다.

Google domains에 들어가서 도메인을 선택하면 왼쪽 탭이 아래와 같이 뜨는데, 여기서 "이메일"탭을 선택하세요.

![Google domains의 Email page](/images/custom-domain-email/google_domains_email_page.png)

여기서 가장 하단에 보시면 Email forwarding을 설정하실 수 있습니다. 

아래처럼 새로 추가할 주소, 실제로 수신할 주소를 설정하시면 됩니다.

![Google domains email forwarding setup](/images/custom-domain-email/email_forwarding.png)

설정 이후에 인증 과정이 필요할 수 있습니다.

위의 설정을 마치시면 자동으로 mx record 가 생성됩니다.

이 과정에 대한 자세한 설명은 [Google Domains 의 문서](https://support.google.com/domains/answer/3251241?hl=ko#emailForwarding) 를 참고하세요.

이후 테스트를 해보면 이메일이 아래와 같이 정상 수신됩니다.

![Received test](/images/custom-domain-email/gmail_well_received.png)

# 발신

발신은 좀 복잡합니다.

발신 설정은 크게 아래 단계로 나뉩니다.

1. 이메일 서비스를 등록하기 (AWS SES 사용)
2. 보안 설정하기 (AWS SES 사용)
3. 이메일 클라이언트에 이메일 서비스 연결하기 (gmail 사용)

## AWS SES 사용해서 이메일 서비스 등록하기

SES는 simple email service로, 이메일을 수, 발신해주는 서비스인데, 저희는 수신하는 기능은 쓰지 않을 거고, 발신만 할 겁니다.

### 도메인 등록

먼저 SES에 들어가서 도메인을 등록합시다. 좌측 메뉴에서 Identity Management > Domains로 들어가시면 됩니다.

![SES new domain](/images/custom-domain-email/ses_new_domain.png)

여기서 Verify a new domain을 클릭하시고, 도메인을 입력하시면 됩니다. DKIM을 지금 만드셔도 되는데, 저는 글의 순서 때문에 일단 지금은 하지 않겠습니다. 도메인을 입력하시면 아래와 같은 창이 뜹니다.

![SES domain verification](/images/custom-domain-email/ses_domain_verification.png)

저희는 SES로 이메일을 수신하지 않기 때문에 하단의 mx record는 무시하시면 되고, 상단의 verification 용 TXT record만 Registrar 에 추가해주시면 됩니다.

![SES domain verification](/images/custom-domain-email/ses_domain_verification_registrar.png)

추가하시고 DNS가 propagate 될 때까지 기다리시면 됩니다. 보통 48시간을 이야기하는데, 저는 몇 분안에 됐습니다. 완료되면 Domain page를 새로고침해도 업데이트 되고, AWS에 등록한 이메일로도 알림이 옵니다.

### 이메일 등록

이제 이메일을 등록하시면 됩니다. 저는 me@charo.dev 를 등록해보겠습니다. 좌측 메뉴에서 Identity Management > Email Addresses로 들어가시면 됩니다. Verify a new email address를 클릭하시고, 제가 추가할 이메일인 me@charo.dev를 입력하겠습니다.

![SES email verification](/images/custom-domain-email/ses_email_verification.png)

이후 verify this email address 를 클릭하시면 me@charo.dev 로 확인 메일이 오고, 안의 링크를 클릭하시면 verification이 끝납니다.

### SMTP credential 생성

이메일을 전송하려면 이메일 클라이언트에서 SMTP protocol을 써서 Amazon SES에 말을 걸게되고, Amazon SES는 이메일 클라이언트가 지정해준 곳으로 이메일을 전송하게 됩니다. 이 과정을 위해 SMTP credential 이 필요한데요, Email Sending > SMTP settings로 들어가셔서 생성하시면 됩니다.

![SES smtp credential creation](/images/custom-domain-email/smtp_credential.png)

이 화면에서 Create My SMTP Credentials 를 클릭하셔서 생성하시면 되는데, 한 번 생성한 뒤에는 다시는 정보를 볼 수 없기 때문에 재사용할 예정이시면 안전한 곳에 id/password 를 기록해두실 필요가 있습니다. 

설정으로 넘어가시면 IAM 서비스로 연결되고, 유저네임만 정하시면 곧바로 credential 이 생성됩니다.

![SES smtp credential creation](/images/custom-domain-email/smtp_credential_success.png)

SMTP username 과 SMTP password를 알아야 이메일을 발신 할 수 있습니다. 유출되지 않게 안전하게 보관하세요.

### Sandbox 탈출하기

이제 이메일을 (안전하지 않게) 보낼 준비는 완료되었지만, SES 정책상 sandbox에서 벗어나야 합니다. SES를 이용해서 스팸 메일 등을 발송하는 걸 막기 위해 SES를 처음 사용하는 사용자들은 외부로 이메일을 발송할 수 없는 sandbox에 갇혀있는데요, 이 sandbox에서 벗어나려면 고객센터로 요청을 보내야합니다. 

Email Sending > Sending statistics > Additional Information > Request Increased Sending Limits 을 통해 요청하시는 등 방법을 통해 sandbox에서 탈출하셔야합니다.

약관에 위배되지 않고 잘 사용할지, bounce에 어떻게 대응할지를 물어보는데, 저는 약관 잘 지키겠습니다, 스팸메일 안보내겠습니다. Bounce가 나면 조치하겠습니다 (보통 newsletter 를 많이 보내니까 bounce가 나면 subscription에서 해당 주소를 없애야겠죠. 그치만 저희는 newsletter를 보내는 게 아니니까요.) 이렇게 썼고, 24시간을 꼬박 기다리니까 (반드시 24시간 이내에 답변이 옵니다.) 승인이 났습니다.

## 보안 설정하기

위에서 말씀드린대로 현재 설정만 사용하면 이메일을 송신할 수는 있지만, spoofing 에 아주 취약한 상태가 됩니다. SPF, DKIM, DMARC 중 하나도 설정되어 있지 않아서 그런데요, 하나 하나 설정해보겠습니다. 

제가 아는 한에서 설명하면 SPF는 이메일을 SES를 통해서만 보낼 거니까 SES에서 오지 않은 이메일이 내 도메인에서 왔으면 그건 사기야! 라고 하는거고, DKIM을 설정하면 이메일 본문에 DKIM-Signature 헤더가 삽입되고, 수신자 측에서는 DNS에서 안내하는 public key로 이 헤더를 복호화해서 이메일 인증에 사용합니다. DMARC는 정책으로, SPF와 DKIM 테스트를 실패하면 어떻게 처리할지 안내합니다. (정책에 실패하면 스팸으로 보내라든지, 아예 수신을 거부한다든지..) SPF나 DKIM이 제대로 설정되지 않은 상태에서 DMARC를 설정하면 모든 메일이 거절되든지 스팸처리가 되니 순서를 지켜서 SPF, DKIM을 먼저 설정을 하셔야합니다. 또, SPF나 DKIM을 설정하시더라도 DMARC가 설정되지 않으면 메일 수신자측에서 딱히 행동을 하지 않는 경우도 있어 spoofing에 그대로 노출되실 수 있으니 꼭 SPF, DKIM, DMARC를 모두 설정하시기 바랍니다.

### SPF

```"v=spf1 include:amazonses.com ~all"```
라는 TXT 레코드를 추가해주시면 끝납니다. 자세한 설명은 [SES 공식 문서](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-authentication-spf.html)를 참고하세요.

제가 이해하기로는 SPF를 설정하면 이 이메일이 실제로 SPF로 설정한 주소에서 오는지 확인하는 과정을 추가로 거치게 됩니다. 이 과정에서 일부 spoofing을 방지할 수 있지만, dkim과 함께 쓰는 게 일반적입니다.

### DKIM

아까 건너뛴 DKIM 설정입니다. 아까 하셨으면 건너뛰시면 됩니다.

AWS SES > Identity Management > Domains > 본인 도메인 (저의 경우는 charo.dev) 로 들어갑니다.

하단의 DKIM 설정으로 들어가셔서 Generate DKIM Settings 를 클릭합니다.

![SES dkim creation](/images/custom-domain-email/ses_dkim.png)

CNAME 세개가 보일 겁니다. Registrar에 세개를 추가해주시고 DNS propagation을 좀 기다려주시면 끝납니다.

자세한 설명은 역시 [공식 문서](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-authentication-dkim.html) 를 참고하세요.

### DMARC

SPF와 DKIM은 스스로도 어느 정도 보안 기능을 해주지만, DMARC를 설정하기 위한 포석입니다. DMARC는 아래처럼 생겼는데요,

```
v=DMARC1;p=reject;pct=100;rua=mailto:postmaster@dmarcdomain.com
```
위를 해석하면 v(버전)는 DMARC1, p(테스트 실패시 정책, policy)는 reject(스팸메일에도 넣지 말고 아예 거절), pct(percentage)는 DMARC 정책을 적용할 정도, 여기서는 100%(모든 이메일에 적용)로 설정됐고요, rua는 feedback 수신자입니다. 모든 태그에 대한 자세한 설명을 읽고 싶으시면 [rfc7489](https://tools.ietf.org/html/rfc7489) 의 섹션 6.3을 읽어보시면 됩니다.

[DMARC 공식 사이트[(https://dmarc.org/overview/)에서는 아래와 같이 설정하라고 권장합니다.

```
1, Deploy DKIM & SPF. You have to cover the basics, first.
2. Ensure that your mailers are correctly aligning the appropriate identifiers.
3. Publish a DMARC record with the “none” flag set for the policies, which requests data reports.
4. Analyze the data and modify your mail streams as appropriate.
5. Modify your DMARC policy flags from “none” to “quarantine” to “reject” as you gain experience.
```
1번은 DKIM & SPF를 먼저 설정하라는 건데, 저희는 마쳤고요, 2번 역시 가이드를 잘 따르셨다면 (테스트를 해보셔도 되고요) 만족할 겁니다. 3, 4, 5가 중요한데요,
3번에서는 처음에는 DMARC를 추가하되 `p=none`(DMARC 테스트가 실패하더라도 아무 행동을 하지 않음) 으로 배포해서 먼저 DMARC test를 통과하는지 확인하고, 4번에서 확인한 결과 문제가 있으면 수정하고, 5번에서는 잘 되면 `p=none`에서 `p=quarantine`(DMARC 테스트가 실패하면 스팸으로 보냄)으로 수정하고, 또 문제가 없으면 `p=reject`(스팸으로 보내지도 않고 아예 거절)로 올리라는 내용입니다.

위 권장 사항을 따르면 처음 DMARC는
```
v=DMARC1;p=none;pct=100;rua=mailto:postmaster@dmarcdomain.com
```
의 형태가 되겠고요, (rua는 리포트를 수신하실 이메일로 바꿔주셔야합니다.)

잘 되면 
```
v=DMARC1;p=quarantine;pct=100;rua=mailto:postmaster@dmarcdomain.com
```
으로 바꾸고, 이 설정에서도 문제가 없으면 최종적으로
```
v=DMARC1;p=reject;pct=100;rua=mailto:postmaster@dmarcdomain.com
```
으로 올리시면 됩니다.

저는 귀찮아서 그냥 처음부터 reject로 했습니다. 다만 이렇게 reject로 설정하면 수신자가 이메일을 받지 못하는 상황이 올 수도 있습니다.

아무튼 이런 형태의 DMARC를 TXT 레코드로 도메인에 추가해주시면 됩니다. KEY는 \_dmarc.(주소)가 됩니다. 저의 경우에는 `_dmarc.charo.dev`가 key, value는 `v=DMARC1;p=reject;pct=100;rua=mailto:{제 이메일 주소}`로 설정했습니다.

자세한 사항은 역시 [SES의 공식 문서](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-authentication-dmarc.html) 를 참고하세요.

이제 모든 설정이 끝났습니다. 이메일 클라이언트에 연결만 해주면 이메일을 송신할 수 있습니다.

## 이메일 클라이언트에 연결하기

이메일을 보내려면 클라이언트에서 송신을 해야겠죠. 두 클라이언트로 설정하는 법을 보여드리겠습니다.

하나는 Gmail, 나머지는 Apple Mail입니다.

### Gmail
Gmail은 톱니바퀴 > 설정에 들어가셔서

![Gmail settings](/images/custom-domain-email/gmail_settings.png)

계정 및 가져오기 탭에서

![Gmail accounts](/images/custom-domain-email/gmail_accounts.png)

다른 주소에서 메일 보내기 > 다른 이메일 주소 추가를 클릭하시면 됩니다.

![Gmail send as](/images/custom-domain-email/gmail_send_as.png)

이제 smtp를 설정하게 되는데요, 발송할 메일 주소 (저의 경우에는 me@charo.dev)를 추가하고 다음 단계로 넘어갑니다.

![Gmail smtp 1](/images/custom-domain-email/gmail_smtp_2.png)

이제 아까 만들어둔 credential 정보를 입력하면 끝납니다.

smtp서버는 SES > smtp settings에서 확인하실 수 있고요 (제 경우는 email-smtp.us-west-2.amazonaws.com). 사용자 이름과 비밀번호는 "SMTP credential 생성" 섹션에서 잘 저장해놓으라고 한 걸 여기서 입력하시면 됩니다.

그러면 이메일로 확인 링크가 오고, 확인 링크를 클릭하시거나 verification token을 입력하시면 끝납니다.

이제 gmail로 이메일을 보내실 수 있습니다.

### Apple Mail

Apple mail도 똑같이 smtp로 설정합니다.

Mail > Preference 로 들어가셔서

![apple mail pref](/images/custom-domain-email/apple_pref.png)

먼저 이메일 주소를 추가합니다. Account Information 항목에서 Email address 항목에서 추가해주시면 되고요,

![apple add addr](/images/custom-domain-email/apple_add_address.png)

이제 Server settings 항목에서 edit smtp list로 들어갑니다.

![apple mail edit smtp](/images/custom-domain-email/apple_smtp.png)

+버튼을 눌러서 새로운 smtp 항목을 추가하고요

![apple mail add smtp](/images/custom-domain-email/apple_smtp_add.png)

gmail에서 하셨던 것처럼 서버, 계정, 비번을 넣어주시면 됩니다. 

다만 여기는 추가작업이 필요한데요, Automatically manage connection settings의 체크를 해제해주시고, Authentication에서 password를 선택해주시면 됩니다. 

![apple mail auth](/images/custom-domain-email/apple_authentication.png)

이러면 모든 설정이 끝이 납니다.

# 테스트

이제 모든 게 잘 동작하는지 테스트해볼 차례입니다. 제가 애용하는 사이트는 [Mail Tester](https://www.mail-tester.com/) 인데요, 여기로 접속하시면 아래처럼 임시 주소가 하나 있습니다.

![mail tester front](/images/custom-domain-email/mail_tester_front.png)

이 임시주소로 이메일을 보내면 저희가 지금까지 설정한 SPF, DKIM, DMARC 등을 검증해줍니다. 중요하진 않지만 제목과 내용도 점수에 들어가서 저는 보통 의미없는 뉴스기사 등을 복붙해서 이메일 내용을 채웁니다. 지금은 이 글 내용을 본문으로 해서 메일을 보내보겠습니다.

점수 확인을 누르면 아래처럼 점수가 보이는데요, 이 글을 그대로 따라하셨다면 (저는 그렇게 했으니까요) 만점이 나올겁니다.

![mail tester score](/images/custom-domain-email/mail_tester_score.png)

중요한 항목은 "You're properly authenticated" 항목입니다. 이 항목을 보시면 SPF, DKIM, DMARC 등 설정을 확인하실 수 있습니다.

아래처럼 전부 체크가 돼있으면 문제가 없는 겁니다.

![mail tester auth](/images/custom-domain-email/mail_tester_auth.png)

고생하셨습니다!