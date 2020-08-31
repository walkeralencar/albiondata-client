我是光年实验室高级招聘经理。
我在github上访问了你的开源项目，你的代码超赞。你最近有没有在看工作机会，我们在招软件开发工程师，拉钩和BOSS等招聘网站也发布了相关岗位，有公司和职位的详细信息。
我们公司在杭州，业务主要做流量增长，是很多大型互联网公司的流量顾问。公司弹性工作制，福利齐全，发展潜力大，良好的办公环境和学习氛围。
公司官网是http://www.gnlab.com,公司地址是杭州市西湖区古墩路紫金广场B座，若你感兴趣，欢迎与我联系，
电话是0571-88839161，手机号：18668131388，微信号：echo 'bGhsaGxoMTEyNAo='|base64 -D ,静待佳音。如有打扰，还请见谅，祝生活愉快工作顺利。

[![CircleCI](https://circleci.com/gh/broderickhyman/albiondata-client/tree/master.svg?style=svg)](https://circleci.com/gh/broderickhyman/albiondata-client/tree/master) [![Go Report Card](https://goreportcard.com/badge/github.com/broderickhyman/albiondata-client)](https://goreportcard.com/report/github.com/broderickhyman/albiondata-client)

# Albion Data - Client
Distributed client for the [Albion Online Data](https://www.albion-online-data.com/)
project.

A quick note on the legality of this application and if it
violates the Terms and Conditions for Albion Online. Here is
the response from SBI when asked if we are allowed to do
monitor network packets relating to Albion Online:
> Our position is quite simple. As long as you just look and
analyze we are ok with it. The moment you modify or manipulate
something or somehow interfere with our services we will react
(e.g. perma-ban, take legal action, whatever).

~ MadDave - Technical Lead for Albion Online

Source: https://forum.albiononline.com/index.php/Thread/51604-Is-it-allowed-to-scan-your-internet-trafic-and-pick-up-logs/?postID=512670#post512670

This client monitors local network traffic, identifies UDP packets
that contain relevant data for Albion Online, and ships the information
off to a central NATS server that anyone can subscribe to.

[Client download stats](https://www.somsubhra.com/github-release-stats/?username=broderickhyman&repository=albiondata-client)

### Contributing
This process is run on a [DigitalOcean Droplet](https://www.digitalocean.com) in order to ensure almost perfect uptime and high performance for the users. If you find this project beneficial to you then please consider a donation, thanks!!

[Become a Patron on Patreon!](https://www.patreon.com/bePatron?u=10422119)

[![ko-fi](https://www.ko-fi.com/img/donate_sm.png)](https://ko-fi.com/E1E5K69V)

# Contributions
Many thanks to the original developers:
- [Regner](https://github.com/Regner)
- [pcdummy](https://github.com/pcdummy)
- [Ultraporing](https://github.com/Ultraporing)

# Downloads
Downloads can be found here: https://github.com/broderickhyman/albiondata-client/releases

## Running on Mac

1. Download the latest `update-darwin-amd64.gz` file from [the Releases page](https://github.com/broderickhyman/albiondata-client/releases)
2. Unzip that file from the Finder or with `gunzip update-darwin-amd64.gz`
3. The unzipped `albiondata-client` file is a Golang binary file. You'll need to make this file executable so it can be run directly. You can do this from your Terminal with: `chmod +x albiondata-client`
4. Run the client from your Terminal with `./albiondata-client`

# Related Projects
- [albiondata-deduper-dotNet](https://github.com/BroderickHyman/albiondata-deduper-dotNet)
- [albiondata-sql-dotNet](https://github.com/BroderickHyman/albiondata-sql-dotNet)
- [albiondata-api-dotNet](https://github.com/BroderickHyman/albiondata-api-dotNet)
- [AlbionData.Models](https://github.com/broderickhyman/albiondata-models-dotNet) [![NuGet](https://img.shields.io/nuget/v/AlbionData.Models.svg)](https://www.nuget.org/packages/AlbionData.Models/)
- [albion-data-website](https://github.com/broderickhyman/albion-data-website)

# Contact Us
The best way to get in touch with us is on the Albion Online Fansites Discord server in either the #proj-albiondata or the #developers channel. A permanent invite link can be found here: [https://discord.gg/TjWdq24](https://discord.gg/TjWdq24)

# Developer Setup
### Mac/Linux Setup
- Install [Dep](https://github.com/golang/dep)
  - Any OS: `go get -u github.com/golang/dep/cmd/dep`
  - Mac with Homebrew: `brew install dep`
- Install dependencies using `dep ensure`

### Windows Setup
[Windows Setup Guide](https://github.com/broderickhyman/albiondata-client/wiki/Building-in-Windows)

# License
This project, and all contributed code, are licensed under the MIT
License. A copy of the MIT License may be found in the repository.
