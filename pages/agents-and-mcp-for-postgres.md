---
title: Agents and MCP for Postgres
timestamp: 2026-09-10T11:30:01
author: szabgab
published: true
description:
tags:
    - Postgres
    - AI
---

## Description

AI agents need more than just prompts - they work with tools and data. This presentation explores the evolution of agentic workflows and the specific mechanisms they use to interact with databases. We’ll focus on the Model Context Protocol (MCP) and how it connects AI to Postgres, moving beyond basic RAG to more advanced data driven tools.

While RAG (Retrieval-Augmented Generation) is well-understood, it is often a one-way street. The next frontier is agents allowing models to interact with data, query, and reason over databases dynamically. The Model Context Protocol (MCP) is an industry standard that solves the fragmentation of tool-calling. This talk provides a practical, "how-to" guide for the popular open-source tools in that field and how they interact with PostgreSQL.

## Bio

[Gleb Otochkin](https://www.linkedin.com/in/glebotochkin/)

Gleb, a Cloud Advocate at Google, specializes in database technologies and AI integration within data-driven applications. His expertise includes relational databases, data transformation and ETL, application development, data replication and integration solutions. He is a runner completing 2-4 marathons per year. In the past he participated in expeditions dedicated to oceanic research in the Pacific as oceanologist.

## Length

60 min

{% youtube id="-ww_b3xS7Rw" file="2026-09-09-agents-and-mcp-for-postgres-with-gleb-otochkin.mp4" %}

## Transcript

1
00:00:01.820 --> 00:00:18.310
Gabor Szabo: So, hello and welcome to this presentation, to the Code Maven channel, if you are watching the video. My name, Gates Gabor. I run these events. Thank you for everyone who arrived to this presentation, live presentation. It's always fun to have

2
00:00:18.310 --> 00:00:26.479
Gabor Szabo: audience in the presentation, and thank you very much, Gleb, for agreeing to give this presentation. In a second, I'll have the

3
00:00:26.480 --> 00:00:33.579
Gabor Szabo: I'll give you the microphone, or the stage, or whatever, to introduce yourself and give the presentation.

4
00:00:33.580 --> 00:00:54.070
Gabor Szabo: Those people who are… if you're watching this in YouTube, then please start by liking the video and following the channel, and remember, below the video, you will find links to notes about the presentation, and links to further future events, where you can follow them and see what might be interesting for you.

5
00:00:54.940 --> 00:01:07.530
Gabor Szabo: I think that's enough for the first introduction, and Glebi will have the chance to introduce yourself and talk about the subject, so thank you very much again for…

6
00:01:07.780 --> 00:01:23.620
Gabor Szabo: coming. Oh, one more thing, those people who are here, that's the perk of being in the live presentation, you can ask questions, and we'll connect them, I guess, and once in a while, Gleb will answer the questions he likes.

7
00:01:23.970 --> 00:01:24.930
Gabor Szabo: Okay?

8
00:01:25.160 --> 00:01:26.190
Gleb Otochkin: Right.

9
00:01:27.260 --> 00:01:41.980
Gleb Otochkin: I'm not going to answer the questions I don't like, or that… no, no, of course not. All right. So, my name is Gleb. We are starting, to talk about agents, MCP, and Postgres.

10
00:01:42.040 --> 00:01:49.500
Gleb Otochkin: database, how we connect to each other, what agent, what MCP, and so on. My name is Gleb.

11
00:01:49.690 --> 00:02:04.380
Gleb Otochkin: I live in Ottawa, Canada. I used to be a scientist in astronaut, participating in some Pacific research expeditions, sometimes spending, like, 6 months in the sea.

12
00:02:04.380 --> 00:02:12.630
Gleb Otochkin: But, latest switch to software engineering and database management. So, I do a lot of running.

13
00:02:12.810 --> 00:02:22.180
Gleb Otochkin: I did a couple of marathons this year, Boston and Ottawa, and next month, I think this October…

14
00:02:22.290 --> 00:02:30.020
Gleb Otochkin: 10th, I'm running Lisbon, so, if you are in Lisbon, let's grab a coffee next month.

15
00:02:30.870 --> 00:02:42.970
Gleb Otochkin: I work at Google as a cloud advocate for databases. What it means, I'm kind of providing the bridge between developers' community and product team inside Google, trying to

16
00:02:43.020 --> 00:03:00.010
Gleb Otochkin: from one side, deliver the message from developer community about the product, and from other side, deliver some kind of best way how to use product to developer community. That's my role, and I think it is a lot of fun, doing that with people.

17
00:03:00.890 --> 00:03:16.219
Gleb Otochkin: So, there are some agenda from high level. We are going to talk about what agents, what MCP, how it works, then we talk about how they are connected to each other. This is how agents use Postgres, and then I'm…

18
00:03:16.220 --> 00:03:24.449
Gleb Otochkin: I want to talk about some tools, like MCP Toolbox, how it works, and what the benefit of the tool,

19
00:03:24.920 --> 00:03:42.799
Gleb Otochkin: Also, I would like to touch a little bit about remote MCP, what we have in the cloud, at Google Cloud, and we talk about NL to SQL, natural language to SQL, and hopefully, if time permits, I will show some simple demo, how it works in real life.

20
00:03:42.900 --> 00:03:51.510
Gleb Otochkin: Based on, agent, MCP, Toolbox, and Postgres database, alright? So, let's start.

21
00:03:52.960 --> 00:03:55.849
Gleb Otochkin: And the first part is,

22
00:03:56.090 --> 00:04:05.550
Gleb Otochkin: I probably… you already know what the agent is, but I still need to touch base. And the agent is… it is kind of…

23
00:04:05.890 --> 00:04:06.890
Gleb Otochkin: AI?

24
00:04:07.580 --> 00:04:12.850
Gleb Otochkin: driven application. It means the application is managed

25
00:04:13.130 --> 00:04:19.740
Gleb Otochkin: and reasoning by AI model, and it has some tools in possession.

26
00:04:19.940 --> 00:04:33.310
Gleb Otochkin: and disposal to apply some actions to real world. In case of real world, what I mean is the tool can, for example, do something on your computer.

27
00:04:33.410 --> 00:04:48.340
Gleb Otochkin: tool can do something on remote computer, and tool do… can do some action. So, technically, we have key components. It is model, tools, and inside agent, you have orchestration, which is agent brain, with

28
00:04:48.390 --> 00:04:57.390
Gleb Otochkin: Goals, instructions, memory, short-term, long-term, and everything else, with guardrail and so on. That's the agent.

29
00:04:57.880 --> 00:05:00.570
Gleb Otochkin: So, how Agent works in…

30
00:05:00.740 --> 00:05:12.760
Gleb Otochkin: very simple case. You have an agent which has in possession, for example, a reasoning tool, and acting tool. And, for example, we have agent which has,

31
00:05:13.070 --> 00:05:21.980
Gleb Otochkin: Access to weather services, And it has access to some kind of alarm signal for…

32
00:05:22.150 --> 00:05:32.990
Gleb Otochkin: user. In that case, what I am asking, set my alarm when the weather is right for running. I don't specify exactly what the weather is right for running.

33
00:05:33.540 --> 00:05:37.810
Gleb Otochkin: And I don't specify exactly where I am.

34
00:05:37.970 --> 00:05:52.060
Gleb Otochkin: But, assuming that agent keeps in memory that I am in Ottawa, and it has access to weather in Otolera, and maybe it has some kind of information about my habits and everything else.

35
00:05:52.240 --> 00:06:05.470
Gleb Otochkin: I probably have very good chance to get proper actions at the end. So, agent will check the weather in Ottawa. When it is sunny and agent likes.

36
00:06:05.470 --> 00:06:17.019
Gleb Otochkin: Agent knows that I like to run in sunny weather, and then it put alarm, okay, it is sunny outside in Ottawa, go for a run. That's the agent work.

37
00:06:17.330 --> 00:06:36.799
Gleb Otochkin: That is probably a silly and very simplified example, but it is what exactly agent is. It uses the model to go through workflow, define how it is going to execute the tools to achieve the goal. That's the agent in natural.

38
00:06:38.270 --> 00:06:44.180
Gleb Otochkin: The problem comes when we have multiple agents, And multiple tools.

39
00:06:44.420 --> 00:06:52.950
Gleb Otochkin: And we had… I was helping to work with some different implementations, And sometimes you have…

40
00:06:53.450 --> 00:07:01.739
Gleb Otochkin: tens of agents. Sometimes you have hundreds of agents. You have tens or hundreds of tools. And the problem is.

41
00:07:01.850 --> 00:07:19.790
Gleb Otochkin: The tool and agents can be developed by different team, by different framework, by different languages, and everything else, and it means for each agent, you have to define some kind of adapter, how agent going to call that particular tool, right?

42
00:07:19.790 --> 00:07:39.210
Gleb Otochkin: And, for example, how to call the weather services. You can create functions in your Perl or Python language code, or Go language code, how to call API of the weather service, but that tool will be only for that particular

43
00:07:39.280 --> 00:07:41.390
Gleb Otochkin: particular agents.

44
00:07:41.610 --> 00:07:58.429
Gleb Otochkin: When you develop it outside of agent and define some kind of protocol, how you connect to that tool, it means you have to create the same adapter for other agents as well. So, that creates tons of different adapters and tons of different implementations.

45
00:08:00.030 --> 00:08:07.179
Gleb Otochkin: It is a problem with scaling and interoperability. And…

46
00:08:07.280 --> 00:08:21.700
Gleb Otochkin: Anthropic, I believe it was Anthropic, came the first with idea. Let's put some kind of standards to that, and let's create an open protocol with standards how applications provide context to LLM.

47
00:08:22.090 --> 00:08:32.340
Gleb Otochkin: And… It is how model context protocol was invented. And, of course, everybody was Decided, oh, that's…

48
00:08:32.570 --> 00:08:43.289
Gleb Otochkin: Great idea, let's embrace it. And we, as a company, Google, and other companies created some kind of body which is trying to

49
00:08:44.570 --> 00:08:58.970
Gleb Otochkin: community-based, open-source community to try to define the standards, and keep the standards, and update the standards based on latest information, developers, development, and everything else.

50
00:08:59.100 --> 00:09:10.649
Gleb Otochkin: And you can read about everything about that. It is modelcontextProtocol.ai, where all the information, all the definition of the standards are.

51
00:09:11.560 --> 00:09:12.990
Gleb Otochkin: So,

52
00:09:13.240 --> 00:09:25.990
Gleb Otochkin: that, in essence, what we replaced. We replaced the A multiplied by T problem to more manageable A plus T problem, which is

53
00:09:26.280 --> 00:09:29.119
Gleb Otochkin: Much more simple to solve.

54
00:09:29.510 --> 00:09:35.539
Gleb Otochkin: And… the MCP itself developed over the years.

55
00:09:35.940 --> 00:09:45.569
Gleb Otochkin: So, the old classic MCP until this summer was stateful, based on JSON RPC,

56
00:09:45.740 --> 00:09:57.100
Gleb Otochkin: two zero messages, and in a nutshell, what you have is you have MCP host, where you place that MCP client, the protocol implementation.

57
00:09:57.430 --> 00:09:59.909
Gleb Otochkin: Then, you use a transport

58
00:10:00.230 --> 00:10:13.270
Gleb Otochkin: standard AO or HTTPSC to get the MCP server environment, which has different resources, tools, prompts, and everything else in possession, which

59
00:10:13.410 --> 00:10:16.700
Gleb Otochkin: can connect to external systems. So.

60
00:10:17.140 --> 00:10:25.020
Gleb Otochkin: it helps, you know how to send the request to MCP server, you know how to get in response.

61
00:10:25.130 --> 00:10:31.129
Gleb Otochkin: And the session ID is persistent during your interaction with MCP.

62
00:10:31.280 --> 00:10:44.960
Gleb Otochkin: So, essentially, what you do, the phase one, it is you discover what kind of tools you have, you load that tools to your memory, and then you use that tools to call the tools with parameters

63
00:10:44.960 --> 00:10:53.309
Gleb Otochkin: to get that inform… provide that information to MCP, and MCP server will execute tools with your parameter.

64
00:10:53.310 --> 00:10:56.249
Gleb Otochkin: And then get a result to you.

65
00:10:56.270 --> 00:11:02.759
Gleb Otochkin: During the execution, it can request additional information from you, and in that case.

66
00:11:02.870 --> 00:11:09.310
Gleb Otochkin: It is two-way communication. It is not only one way you get your HTTP post.

67
00:11:09.720 --> 00:11:20.900
Gleb Otochkin: pass that information, and that's it. No. You are getting in response, acknowledgement, and probably request for another information, piece of information, you pass again, and then you get a result.

68
00:11:21.370 --> 00:11:22.629
Gleb Otochkin: That is great.

69
00:11:22.750 --> 00:11:25.419
Gleb Otochkin: And… but it is not scalable.

70
00:11:25.690 --> 00:11:35.880
Gleb Otochkin: If you have… very high loaded, like, hundreds of agents and tens of MCP servers behind the scenes.

71
00:11:37.000 --> 00:11:41.080
Gleb Otochkin: And you want to be it… Scalable enough

72
00:11:41.210 --> 00:11:44.500
Gleb Otochkin: to increase number of ports for MCP servers.

73
00:11:45.250 --> 00:11:50.590
Gleb Otochkin: and place it behind the load balancer, it's not gonna work, really.

74
00:11:50.940 --> 00:11:54.120
Gleb Otochkin: And it is one of the reasons why,

75
00:11:54.510 --> 00:12:00.080
Gleb Otochkin: new MCP standards were adopted recently, it is this summer.

76
00:12:00.250 --> 00:12:03.319
Gleb Otochkin: Now it is stateless NCP client.

77
00:12:03.530 --> 00:12:07.459
Gleb Otochkin: and stateless replica of MCP servers.

78
00:12:07.980 --> 00:12:26.740
Gleb Otochkin: it can work behind the load balancer. What is replace? So, when you pass information to MCP Server, you actually put additional information underscore meta, where you put all information about client, some,

79
00:12:26.830 --> 00:12:31.040
Gleb Otochkin: housekeeping information for your post and everything else, but each

80
00:12:31.190 --> 00:12:40.860
Gleb Otochkin: communication. You post, you get results, it's self-sufficient, so you don't need to keep that sticky session anymore. And

81
00:12:41.080 --> 00:12:45.460
Gleb Otochkin: As a matter of fact, session ID was dropped from the standard itself.

82
00:12:45.660 --> 00:12:48.390
Gleb Otochkin: So, that was what changed.

83
00:12:48.730 --> 00:13:06.149
Gleb Otochkin: And of course, phase one, where discovery and edge cache capabilities, is completely optional for communication, because it might be already cached somewhere, for example, on edge stuff, all the tools and everything else, or it is

84
00:13:06.860 --> 00:13:25.200
Gleb Otochkin: you can just download that Prakash tools and use that tool. You know what kind of requests to pass, you know how to communicate and everything else. So, that's the main replacement. Of course, we… if you go to the site, you can read much more,

85
00:13:25.270 --> 00:13:33.870
Gleb Otochkin: Detailed description, what exactly has changed during the communication, and what exactly is on the roadmap.

86
00:13:34.130 --> 00:13:39.679
Gleb Otochkin: So, that's how MCP, in general, works.

87
00:13:41.670 --> 00:13:49.189
Gleb Otochkin: Couple of… few… really few, practical, notes about MCP itself.

88
00:13:50.500 --> 00:13:58.990
Gleb Otochkin: the MCP in general, what I understood from my working… from working with different companies and different developers.

89
00:13:59.460 --> 00:14:18.539
Gleb Otochkin: Yes, MCP can be some kind of resemblance of API, but in reality, what you probably want to build tools around your critical user journey. So, for example, if user is asking, I want to get a report

90
00:14:18.710 --> 00:14:20.869
Gleb Otochkin: For the last month.

91
00:14:21.050 --> 00:14:32.150
Gleb Otochkin: And you know the users are asking that particular case report for last month, for that particular piece of your information, or sales, or anything else.

92
00:14:33.170 --> 00:14:41.139
Gleb Otochkin: What you want in that case, you want to create the tool which will be executed for that intent.

93
00:14:41.650 --> 00:14:48.369
Gleb Otochkin: And… you don't want to provide, okay, you want to report for last month. Here, the…

94
00:14:48.530 --> 00:14:50.939
Gleb Otochkin: schema. Here are the…

95
00:14:51.070 --> 00:15:02.139
Gleb Otochkin: tools like execute SQL statement, check the metadata, and everything else, then you probably can execute several tools and get reports last month.

96
00:15:02.430 --> 00:15:13.819
Gleb Otochkin: That's maybe not the most optimal way, and the reason is, in that case, you spend much more time, and spent much more talking as well.

97
00:15:14.440 --> 00:15:21.089
Gleb Otochkin: So, when you build tools, think about goals and what kind of outcome you want to get.

98
00:15:21.190 --> 00:15:29.390
Gleb Otochkin: The arguments itself, as well, less arguments better for any kind of tool, and

99
00:15:29.490 --> 00:15:42.289
Gleb Otochkin: what we… what at least I see in my experience, when you have multi-layer dictionary-type argument, it doesn't really work very well with the model.

100
00:15:43.100 --> 00:15:46.390
Gleb Otochkin: Then, name tools appropriately.

101
00:15:46.760 --> 00:16:00.480
Gleb Otochkin: Keep in mind, we are still working with large language models. It means they understand language. If you name your tool drop database, but in reality tool is executing SQL statement.

102
00:16:00.790 --> 00:16:08.610
Gleb Otochkin: universal SQL statement. That probably kind of, not intuitive and

103
00:16:08.810 --> 00:16:12.410
Gleb Otochkin: Agent might not pick up that tool to execute the statement.

104
00:16:12.870 --> 00:16:17.680
Gleb Otochkin: But it will try to pick up that tool to drop database.

105
00:16:18.010 --> 00:16:23.630
Gleb Otochkin: If you ask to drop database, and in both cases, it might be not successful.

106
00:16:24.530 --> 00:16:33.979
Gleb Otochkin: Think about context. Every text output log is going to context. Of course, we try to cache context as much as possible, but still.

107
00:16:34.330 --> 00:16:42.410
Gleb Otochkin: More description for your tool. If you decide, I have a tool, and I have a several-page description, what tool does.

108
00:16:43.100 --> 00:16:53.439
Gleb Otochkin: And if you have 10 of such tools in your MCP server, it will clog your contacts and might reduce quantity of execution.

109
00:16:53.950 --> 00:16:58.189
Gleb Otochkin: So, again, and MCP is not API.

110
00:16:58.490 --> 00:17:08.590
Gleb Otochkin: don't make your MCP server exactly resembling your API access to your database management system, to your data, or anything else.

111
00:17:08.700 --> 00:17:23.650
Gleb Otochkin: In some cases, it is justified, but there are only few such cases. I maybe touch-based on some cases later in the presentation, but that's the general advice. It doesn't cover everything, but at least something, right?

112
00:17:24.569 --> 00:17:25.540
Gleb Otochkin: So…

113
00:17:25.770 --> 00:17:33.949
Gleb Otochkin: I'm stopped here, and if we have any kind of questions, let me, let me know. If you…

114
00:17:34.710 --> 00:17:37.159
Gleb Otochkin: If you are with me, let me know as well.

115
00:17:39.890 --> 00:17:42.970
Gabor Szabo: So far, I didn't see any questions. I…

116
00:17:44.810 --> 00:17:49.349
Gabor Szabo: If anyone wants to ask questions, then please either write it in the chat.

117
00:17:49.590 --> 00:17:56.709
Gabor Szabo: Or raise your hand, and then probably I… actually, I can allow you now to unmute yourself if you prefer that one.

118
00:18:02.450 --> 00:18:03.200
Gleb Otochkin: Alright.

119
00:18:03.560 --> 00:18:12.380
Gleb Otochkin: So if we don't have any questions for now, just… if you come up with the questions later, after next chapter, I will stop again, so if you…

120
00:18:12.620 --> 00:18:21.190
Gleb Otochkin: Still thinking and, have some questions, you're welcome to ask. Okay, let me continue,

121
00:18:22.270 --> 00:18:25.560
Gleb Otochkin: to talk about how the agents and Postgres

122
00:18:25.750 --> 00:18:27.709
Gleb Otochkin: Can be connected to each other.

123
00:18:29.820 --> 00:18:30.930
Gleb Otochkin: So, first.

124
00:18:31.450 --> 00:18:41.139
Gleb Otochkin: Postgres database. I know probably you already know what it is, but for people who are not familiar with different database engines.

125
00:18:41.450 --> 00:18:49.830
Gleb Otochkin: It is just a couple of words about Postgres. It is a relational database, it means, it is

126
00:18:50.050 --> 00:18:58.729
Gleb Otochkin: Supports relational data schema when the different data are connected to each other.

127
00:18:59.640 --> 00:19:04.910
Gleb Otochkin: The example… examples of different relational database, it is…

128
00:19:05.180 --> 00:19:11.519
Gleb Otochkin: for example, Oracle, one of the old databases, it is Microsoft SQL Server.

129
00:19:11.520 --> 00:19:36.370
Gleb Otochkin: MySQL, all those databases are relational databases, and in reality, relational database, it is like a Swiss knife of the database. It can run reports for you, it can use different joints to connect different tables to get proper information, and it can support all TP workload, when all TP workload, it is when you

130
00:19:36.370 --> 00:19:41.809
Gleb Otochkin: do a lot of small changes to the database. Think about that, like,

131
00:19:41.980 --> 00:19:47.940
Gleb Otochkin: Backend database for online transaction system for your…

132
00:19:48.580 --> 00:20:03.969
Gleb Otochkin: workshop, or for your store online system. So, that's the Postgres database. It is not as young as you think. In reality, it was originated in 1986,

133
00:20:05.520 --> 00:20:22.930
Gleb Otochkin: in Berkeley, and open source release was in 1997. And, of course, it is not Postgres. When it was originated in 1986, it was Ingress, and if you think about it, now it is 2026,

134
00:20:23.210 --> 00:20:28.920
Gleb Otochkin: You can calculate by yourself, it is 40 years ago, right? So…

135
00:20:29.490 --> 00:20:35.880
Gleb Otochkin: It is really a robust, reliable, data, database.

136
00:20:35.880 --> 00:20:56.049
Gleb Otochkin: with all support for ACID, integrity, consistency, and everything else. And it is highly, highly extensible, so you can create extension for Postgres database and add a new functionality, like PGVector, to work with the vector data type with embeddings in Postgres.

137
00:20:56.350 --> 00:21:10.700
Gleb Otochkin: So, Google has tons of different databases, and we support Postgres or Postgres-compatible database. Cloud SQL for Postgres, essentially the same Postgres as you have in,

138
00:21:10.820 --> 00:21:17.690
Gleb Otochkin: in the community edition, but it is just running on Google infrastructure with management layer around that.

139
00:21:17.800 --> 00:21:24.880
Gleb Otochkin: AloDB fully Postgres-compatible database. It means you have a Postgres database.

140
00:21:25.030 --> 00:21:38.369
Gleb Otochkin: and you want to move to AloDB, you don't need to change any code, but AlloDB has additional features which probably might be helpful or might not be helpful to you, depending on your workload.

141
00:21:38.370 --> 00:21:50.080
Gleb Otochkin: AI integration, columnar engine, and some performance improvements, for highly intensive workload and transactions.

142
00:21:50.170 --> 00:22:01.820
Gleb Otochkin: And we have some Postgres interface for Spaniel database. It is not real Postgres, but it is kind of adapter interface for some Postgres-like queries and actions.

143
00:22:02.260 --> 00:22:03.250
Gleb Otochkin: So…

144
00:22:03.680 --> 00:22:18.220
Gleb Otochkin: how agents use database. The first one of the cases, I think it is quite interesting, and I saw several implementations of that. The database is used for persistent memory of agent.

145
00:22:19.010 --> 00:22:36.409
Gleb Otochkin: The second is, I saw prompt management, resided inside database. It is when the database, is storing different versions of prompt, and they, probably system instructions, everything else, and then used

146
00:22:36.590 --> 00:22:42.440
Gleb Otochkin: By the agent to improve or evaluate some different versions of the prompt.

147
00:22:42.600 --> 00:22:49.599
Gleb Otochkin: Rug. Rug stands for, Retrieval Augmented Generation, when the…

148
00:22:49.800 --> 00:23:00.090
Gleb Otochkin: Agent is using information, proprietary or domain-specific information from the database to improve the answer and ground it

149
00:23:00.120 --> 00:23:11.650
Gleb Otochkin: Not based only on open source search or other publicly available information, but also on your proprietary internal information.

150
00:23:12.810 --> 00:23:15.690
Gleb Otochkin: But from other side, Agent Ken.

151
00:23:16.020 --> 00:23:31.440
Gleb Otochkin: be used to manage your database implementation. If your database environment has proper API, and you have developed the tools, MCP server, which can do that to your database. For example, taking backups.

152
00:23:31.840 --> 00:23:46.140
Gleb Otochkin: or check the performance, maybe scale up or scale down, or anything like that. You can use Agent to troubleshoot your database, performance, queries, and everything else, and of course, for monitoring.

153
00:23:46.740 --> 00:23:49.619
Gleb Otochkin: So, let's talk a little bit more about that.

154
00:23:49.870 --> 00:23:59.270
Gleb Otochkin: cases, the persistent memory, in agent. For example, you have daily long interaction with the agent, which

155
00:23:59.510 --> 00:24:03.580
Gleb Otochkin: comes… by days. In reality, Agent

156
00:24:04.750 --> 00:24:13.330
Gleb Otochkin: Tries to keep everything in memory, but different agent frameworks also using some small database of file system storage

157
00:24:13.390 --> 00:24:31.709
Gleb Otochkin: to offload memory from the context, because context is limited for the models, and you have to summarize it from time to time. The compressing context, it is taking a summary of existing context and discarding everything out.

158
00:24:31.910 --> 00:24:36.300
Gleb Otochkin: So, what's the problem with that? The problem is…

159
00:24:36.400 --> 00:24:49.699
Gleb Otochkin: If it is locally deployed, for example, you deploy it on Kubernetes port, your agent, and that works pretty well, you store that information, for example, on,

160
00:24:49.840 --> 00:25:04.249
Gleb Otochkin: file system or SQLite database, which deployed with the agent, but if the port is crushed, and you replace by another port, that information is lost, right? So, what you can do, you can upload it to

161
00:25:04.360 --> 00:25:12.420
Gleb Otochkin: Postgres database, for example, to the tables, and then use Semantic or hybrid search.

162
00:25:12.720 --> 00:25:25.500
Gleb Otochkin: to access that information when user… for example, user is just simple, very silly example. User is asking, hey, do you remember from agent, do you remember,

163
00:25:25.670 --> 00:25:44.309
Gleb Otochkin: I told you 3 weeks ago about my wife's birthday, and about my ideas for the gift. Can you please recall it and tell me if it is still on the sale as it used to be on that time?

164
00:25:44.810 --> 00:26:03.169
Gleb Otochkin: Technically, what you can do in the agent code, you can specify, okay, then go back, use vector search for why gift among the chance of conversation, then retrieve that information, provide to the model context, and then agent knows what the

165
00:26:03.480 --> 00:26:06.619
Gleb Otochkin: talk is about. So that's the idea.

166
00:26:06.850 --> 00:26:17.759
Gleb Otochkin: Also, you can upload the procedural, procedural, part of the, agent. For example, like, rules,

167
00:26:17.840 --> 00:26:31.790
Gleb Otochkin: guardrails, workflows, configuration, in terms of you can store it as JSON data inside the agent, and then use it when the agent loads up to the memory.

168
00:26:31.970 --> 00:26:33.709
Gleb Otochkin: So that's the idea.

169
00:26:34.650 --> 00:26:36.960
Gleb Otochkin: The prompt management, it was…

170
00:26:37.850 --> 00:26:53.439
Gleb Otochkin: It is interesting, I saw at least 3 different cases when people decide, okay, we want to improve our prompt, but we want to improve our prompt based on scientific approach. We don't want anecdotal data. Oh, looks like that prompt provides better results.

171
00:26:53.460 --> 00:27:03.810
Gleb Otochkin: We want to have versioning of the prompt and system instruction inside our database, then execute those prompts

172
00:27:03.850 --> 00:27:11.550
Gleb Otochkin: With our evaluation criterias for hundreds of times, and get appropriate average results

173
00:27:11.650 --> 00:27:20.700
Gleb Otochkin: whether the new versions of prompt is better than old versions of prompt, and so on and so forth. So that actually works really well.

174
00:27:21.070 --> 00:27:30.049
Gleb Otochkin: At least in those cases when people were introducing me to those systems. It was quite interesting.

175
00:27:30.780 --> 00:27:55.299
Gleb Otochkin: I have to mention the rack, it is Retrieval Augmented Generation. It boils down when you have, for example, some kind of proprietary knowledge, your knowledge base inside Postgres database, you create… you split it to the chunks, for example, you put the vector embeddings for each chunk, and then what you do, the consumer, one of the colleagues is asking the system, for example, can you maybe

176
00:27:55.400 --> 00:28:14.609
Gleb Otochkin: provide me a solution for that particular problem. And then, what agent does, agent is, okay, let me check… I'm not going to check in the internet, because you don't have that information in internet. It is your internal problem. It is your internal solution. So, I'm connecting to the database.

177
00:28:14.640 --> 00:28:30.149
Gleb Otochkin: I'm using, for example, vector search, I'm checking that information, I'm retrieving that solution back to my context, then passing context to the model, and then getting the grounded

178
00:28:30.230 --> 00:28:38.739
Gleb Otochkin: best response from the model itself, which is based on your internal proprietary data. That is… Rug in natural.

179
00:28:41.190 --> 00:28:49.510
Gleb Otochkin: So, when you do it on the scale, Sometimes you need to combine Different types of search.

180
00:28:50.010 --> 00:29:07.280
Gleb Otochkin: Plus you want to apply some index on your vector search, for example, or non-vector search, to make it really fast and scalable. And again, we are talking about relational database, and in relational database, you can

181
00:29:07.620 --> 00:29:17.859
Gleb Otochkin: use different filters joins And hybrid search, and data search, and all together to provide the best and most

182
00:29:18.200 --> 00:29:23.769
Gleb Otochkin: performance response. So, It is why

183
00:29:23.990 --> 00:29:40.519
Gleb Otochkin: I think… I can be biased here, of course, and I am probably biased, because I like Postgres. I think Postgres, as a universal database engine, suits very well for such kind of workload.

184
00:29:40.810 --> 00:29:42.770
Gleb Otochkin: You can use…

185
00:29:43.140 --> 00:30:01.979
Gleb Otochkin: consolidated data architecture. You can use all the data types, all variety of the data types, JSON, JSON, B, relational standard data types, like character numbers and everything else, plus additional vector data type, or full-text search, all together.

186
00:30:02.060 --> 00:30:12.300
Gleb Otochkin: to get… retrieve the results relatively quickly. Quick enough for the agent, because what I understand when you work with the agent.

187
00:30:12.930 --> 00:30:17.610
Gleb Otochkin: The database part retrieval of that information is really quick.

188
00:30:18.120 --> 00:30:32.679
Gleb Otochkin: part with the model is taking more time, usually. It is not the most critical part, the database, from retrieval. It is when the people, oh, the Postgres will retrieve my data in 50 milliseconds.

189
00:30:32.830 --> 00:30:38.659
Gleb Otochkin: But if I use… for example, specialized vector database, and can…

190
00:30:38.980 --> 00:30:48.140
Gleb Otochkin: it's limited down to 25 milliseconds, right? And then you wait for 2 seconds to get response from the model. So it is…

191
00:30:48.330 --> 00:30:51.340
Gleb Otochkin: Of course, again, it is definite in each case.

192
00:30:51.620 --> 00:31:02.549
Gleb Otochkin: And another important part, if you have different parts of your information in different databases, for example, vector search and bind database.

193
00:31:02.550 --> 00:31:16.590
Gleb Otochkin: some other type of information, another database. Essentially, you need to combine all those sources together to provide the answer. In that case, you need to create some kind of complicated pipeline to connect them each other, and

194
00:31:16.770 --> 00:31:25.899
Gleb Otochkin: Make it consistent that vector search and the information from non-vector database are consistent among themselves.

195
00:31:26.010 --> 00:31:32.790
Gleb Otochkin: So that creates some kind of problem. The Postgres database is always consistent. Consistent.

196
00:31:33.930 --> 00:31:45.140
Gleb Otochkin: Conversational analytics for database, it is an interesting case when you create an agent which accepts natural language input and

197
00:31:45.490 --> 00:31:59.240
Gleb Otochkin: provide back either a SQL statement, or execute that SQL statement directly inside database, and then provide the results. So… and I will try to show some short demo about that.

198
00:31:59.360 --> 00:32:06.299
Gleb Otochkin: Very basic, but still probably valuable to show the different ways how to do that.

199
00:32:08.830 --> 00:32:19.359
Gleb Otochkin: And management databases. It is already here, it is already here in… some different agents. For example.

200
00:32:19.990 --> 00:32:31.380
Gleb Otochkin: At Google Cloud, you already have ability to manage your databases in MCP servers, because it has that MCP tools available for you. For example, to need

201
00:32:31.990 --> 00:32:47.999
Gleb Otochkin: start, stop your instance, create new instance, create database, for example, I want to backup my database, and MCP Server, if it has the tool, and if you have proper permissions to execute that tool on the backend database.

202
00:32:48.040 --> 00:32:58.229
Gleb Otochkin: It will do it for you. So, it is already here, it is already available. So, it is not something we are talking about tomorrow, it is here, it is now.

203
00:32:59.340 --> 00:33:04.499
Gleb Otochkin: So, I'm stopping here, and if we have any questions, I'm happy to answer.

204
00:33:31.810 --> 00:33:34.080
Gleb Otochkin: Any questions, guys?

205
00:33:39.750 --> 00:33:41.420
Gleb Otochkin: Am I still connecting?

206
00:33:41.740 --> 00:33:45.769
Gabor Szabo: Yeah, yeah, yeah, you're, you're, you're connected, and we hear you.

207
00:33:46.920 --> 00:33:53.479
Gabor Szabo: Either people are overwhelmed, Or you are just answering everything that they are… they would ask.

208
00:33:53.530 --> 00:33:55.610
Gleb Otochkin: Maybe I'm so good.

209
00:33:55.930 --> 00:33:56.670
Gabor Szabo: Yeah.

210
00:33:57.060 --> 00:34:06.369
Gleb Otochkin: I doubt about that, but… okay, if we don't have any questions, let's go forward, how we… about the time?

211
00:34:06.370 --> 00:34:12.419
Gabor Szabo: There was just one comment that says, no questions yet from me, presentation really great and clear so far.

212
00:34:12.920 --> 00:34:14.130
Gleb Otochkin: Great, alright.

213
00:34:14.130 --> 00:34:17.399
Gabor Szabo: There's an expectation that maybe it's not… that's it.

214
00:34:17.409 --> 00:34:27.469
Gleb Otochkin: Right. Okay, let me talk about, MCP toolbox.

215
00:34:27.979 --> 00:34:35.609
Gleb Otochkin: So, we were talking about databases, how they use MCP, what is MCP, what agent. So.

216
00:34:36.059 --> 00:34:40.089
Gleb Otochkin: what kind of tools we have. And here, again,

217
00:34:41.259 --> 00:34:50.299
Gleb Otochkin: I'm always biased, everybody is biased, but our team at Google created a tool, it is an open source tool, it calls MCP Toolbox.

218
00:34:50.659 --> 00:34:58.389
Gleb Otochkin: for databases. It supports Google Database and non-Google database, Equally.

219
00:34:58.549 --> 00:35:07.329
Gleb Otochkin: It means, if you have… Cloud SQL, yes, you can use MCT Toolbox for database, but if you have

220
00:35:08.119 --> 00:35:25.729
Gleb Otochkin: open source Postgres, or MySQL, MySQLite, or Valky, or Elasticsearch, or Apache, Cassandra, or Oracle, you can use MCP Toolbox as well. It is open source, it is available, it has API,

221
00:35:26.109 --> 00:35:39.959
Gleb Otochkin: we have a great team behind the development of MCP Toolbox I'm working closely with. They are publishing a lot of interesting blogs and everything else. It has enhanced authorization support.

222
00:35:40.269 --> 00:35:49.059
Gleb Otochkin: It has SDK, and SDK for goal language, or Python, I believe for Java as well, so…

223
00:35:49.349 --> 00:35:58.419
Gleb Otochkin: You can develop something just using SDK for Toolbox and Toolbox, together. So, that's a great tool.

224
00:35:58.789 --> 00:36:03.639
Gleb Otochkin: And… If you prefer to work with IDEs.

225
00:36:03.879 --> 00:36:13.599
Gleb Otochkin: then, again, you can just… since it is MCP, it is very easy to implement in any kind of tools supporting MCP servers.

226
00:36:13.959 --> 00:36:23.689
Gleb Otochkin: Technically, for example, for… here's the example of configuration for local development. You put the path to MCP toolbox.

227
00:36:23.699 --> 00:36:34.849
Gleb Otochkin: And then you put the arguments how you want the MCP toolbox to start. Here, the argument, it is pre-built for Postgres and standard I.O, it means it is developing

228
00:36:34.939 --> 00:36:52.919
Gleb Otochkin: next to my tool, right? It is, for example, I have Xcode, or I have anti-gravity tool, and I'm just providing that information to my anti-gravity tool. Here, the MCP server you can use, and then you provide Postgres host port database user password.

229
00:36:52.979 --> 00:37:02.249
Gleb Otochkin: And in my case, for example, it is all local database and everything else, and then you are working with database out of box. That's the idea.

230
00:37:02.569 --> 00:37:13.909
Gleb Otochkin: And speaking about the pre-built tools, it means it is universal, some kind of predefined default set of tools you can use with Postgres, but

231
00:37:14.039 --> 00:37:15.709
Gleb Otochkin: Doesn't mean that it's the…

232
00:37:15.829 --> 00:37:32.919
Gleb Otochkin: effective for, for example, for user application, but for development, pre-built tools, probably the most efficient way how to use, because you don't really know, sometimes, what kind of critical user journey you will have when you develop something.

233
00:37:33.469 --> 00:37:46.739
Gleb Otochkin: But also what you can, you can provide the tools and configuration file, and exactly set of tools and tool sets you want to use with your agent as well. And I will show the example later about that.

234
00:37:47.249 --> 00:37:48.209
Gleb Otochkin: So…

235
00:37:48.549 --> 00:37:55.239
Gleb Otochkin: Here, the GitHub repository. It is, again, open source. You go to GitHub repa, you get it to your

236
00:37:55.379 --> 00:38:07.769
Gleb Otochkin: environment, and you use it. And we have documentation, mcptoolbox.dev site, so have a look at how it suits for you, what kind of database engine you use, maybe

237
00:38:07.949 --> 00:38:11.429
Gleb Otochkin: Give it a try, and let us know how it works.

238
00:38:13.589 --> 00:38:22.249
Gleb Otochkin: It is… MCV Toolbox is great for local development, and not only local development for small environments, but

239
00:38:22.369 --> 00:38:31.329
Gleb Otochkin: If you… work… at Google Data Cloud. It is Google Cloud with Google Data Services.

240
00:38:31.479 --> 00:38:40.849
Gleb Otochkin: and you want something more enterprise-ready, you might to look to MCP at Google Cloud. So, essentially what it is.

241
00:38:41.029 --> 00:38:45.009
Gleb Otochkin: It is Google Managed MCP Server.

242
00:38:45.349 --> 00:39:04.709
Gleb Otochkin: And it is MCP Server for different database engines. For example, BigQuery, MCP Server for LoadB, MCP Server for Cloud SQL, MCP Server for other data services. This unified interface, the same protocol for BigQuery, Database Looker, or anything else, is fully managed.

243
00:39:04.799 --> 00:39:10.139
Gleb Otochkin: You don't need to worry about how you're gonna scale it behind the scenes.

244
00:39:10.269 --> 00:39:21.659
Gleb Otochkin: And, built specifically for the reasoning partners on LLM. So, Technically, what you do, you…

245
00:39:22.159 --> 00:39:30.229
Gleb Otochkin: configure MCP server, and then you get a number of tools. And do you remember when I was talking that MCP

246
00:39:30.569 --> 00:39:37.119
Gleb Otochkin: tools are not API. So, here's the case where MCP tool

247
00:39:37.429 --> 00:39:42.009
Gleb Otochkin: might resemble API access, and the reason behind that

248
00:39:42.159 --> 00:39:53.509
Gleb Otochkin: For example, we at Google Cloud, we don't really know what kind of user journey you will have as a customer, right? And when you connect

249
00:39:53.619 --> 00:40:00.899
Gleb Otochkin: We cannot create, as of now, a custom MCP tool on Google Cloud MCP Server for you.

250
00:40:01.769 --> 00:40:17.749
Gleb Otochkin: So, what we provide, we provide some kind of universal tool. Okay, you can, for example, clone instance, you can start instance, you create backup, it is about management your environment at Google Cloud. Also, you can execute

251
00:40:17.919 --> 00:40:37.269
Gleb Otochkin: SQL statement, you can troubleshoot your SQL query, and some… so we provide some basic user journey for you, and then you combine that in your application with intent and everything else, and how they combine different tools together. So that is how it works. Also, it has

252
00:40:37.399 --> 00:40:45.269
Gleb Otochkin: internal data protection. So, apart from the… all the permissions you have to have at Google Cloud.

253
00:40:45.569 --> 00:40:57.879
Gleb Otochkin: it also integrated with model armor, for example, and model armor will help you to prevent you from prompt injection. By the way, that is a real thing. If you tell to

254
00:40:58.189 --> 00:41:03.429
Gleb Otochkin: For example, to your agent, do not

255
00:41:04.289 --> 00:41:11.629
Gleb Otochkin: select from user's table. It is just a wild example. I don't want users' data to be linked.

256
00:41:12.039 --> 00:41:19.979
Gleb Otochkin: It doesn't mean agent will always, refuse to get… Data from users table.

257
00:41:20.369 --> 00:41:24.229
Gleb Otochkin: It means you have to protect it by some more reliable way.

258
00:41:24.509 --> 00:41:26.809
Gleb Otochkin: The prompt injection to get,

259
00:41:27.049 --> 00:41:31.209
Gleb Otochkin: Around such kind of simple guardrail is…

260
00:41:31.529 --> 00:41:40.079
Gleb Otochkin: quite simple. You can ask, for example, any model, create me prompt injection, and it will create it for you.

261
00:41:40.289 --> 00:41:48.739
Gleb Otochkin: Of course, maybe now we have some more guardrail, but again, it is not the big deal to work around such stuff.

262
00:41:49.019 --> 00:41:50.159
Gleb Otochkin: In the…

263
00:41:50.389 --> 00:42:09.589
Gleb Otochkin: MCP Toolbox, you can create also some special… you can start MCP Server with, for example, read-only access to the database, and you can provide special configuration options to prevent write access to the database as well. So, you have different options as well.

264
00:42:10.479 --> 00:42:19.389
Gleb Otochkin: So, it is enabled by default, but it doesn't mean everybody can connect and start using it. You have to provide AM,

265
00:42:19.609 --> 00:42:23.269
Gleb Otochkin: permissions to execute MCP tools.

266
00:42:23.839 --> 00:42:31.899
Gleb Otochkin: And then you have to provide additional permissions to execute different actions on your environment. So, it is not…

267
00:42:32.119 --> 00:42:37.459
Gleb Otochkin: open for everyone, you have to explicitly provide those permissions.

268
00:42:37.569 --> 00:42:56.189
Gleb Otochkin: Your configuration of client is quite simple, it is HTTP URL, it is, for example, for Cloud SQL, it is SQLadmin, Google APIs, com, MCP, and then you provide your Google credentials, you use Google SDK to provide OAuth, or you use API key. So, that's how it works.

269
00:42:57.459 --> 00:43:04.839
Gleb Otochkin: And then you can create table, load, database, schema, run SQL, everything out of box. So… That's…

270
00:43:05.049 --> 00:43:13.289
Gleb Otochkin: MCP server, tools, and what you can have out of box. Google Cloud, and I'm… oh?

271
00:43:15.939 --> 00:43:17.399
Gleb Otochkin: What is happening?

272
00:43:19.529 --> 00:43:20.419
Gleb Otochkin: Oh.

273
00:43:24.069 --> 00:43:26.499
Gleb Otochkin: Sorry, guys, I think it was…

274
00:43:27.429 --> 00:43:35.279
Gleb Otochkin: Gemini decided to do something about, me. All right, natural language…

275
00:43:35.280 --> 00:43:36.660
Gabor Szabo: prompting the injection.

276
00:43:36.880 --> 00:43:43.269
Gleb Otochkin: Yes, it was clear prompt injection. I didn't expect that, for sure. It was not planned.

277
00:43:44.310 --> 00:43:45.250
Gleb Otochkin: So…

278
00:43:45.930 --> 00:44:00.740
Gleb Otochkin: what we have for NL2SQL, of course, I will show a couple of examples NL2SQL a little bit later, but what we developed at Google Cloud, and I think you should try that. It is kind of cool. So.

279
00:44:01.460 --> 00:44:10.509
Gleb Otochkin: By default, when you use Agent and MCP with access to your database, you technically can ask Agent.

280
00:44:11.220 --> 00:44:13.749
Gleb Otochkin: Give me the data, and agents…

281
00:44:14.820 --> 00:44:36.550
Gleb Otochkin: models are smart enough to understand, oh, I have Postgres database, I have access to information schema. Let me check what tables I have. Okay, what's the table's name, what the table's name, columns, and everything else, and technically it can connect to each other and eventually give you the right SQL query and results from the database.

282
00:44:36.550 --> 00:44:37.540
Gleb Otochkin: That works.

283
00:44:37.950 --> 00:44:53.200
Gleb Otochkin: But it is not stable, it takes too much time, especially the first time when it is executed, when it is scanning your information schema and everything else. And next time you ask the same question, you might get different SQL query.

284
00:44:53.310 --> 00:45:08.070
Gleb Otochkin: and that SQL query can be wrong, or a SQL query can be not efficient. For example, you know what SQL query should be executed for that particular intent, for that particular type of equation from the user.

285
00:45:08.420 --> 00:45:13.019
Gleb Otochkin: what you can do with Google Cloud, you can create query data can accept

286
00:45:13.560 --> 00:45:16.499
Gleb Otochkin: And providing query data CAX set.

287
00:45:16.660 --> 00:45:22.870
Gleb Otochkin: Query template, query facets, some additional pieces of information, which helps

288
00:45:23.500 --> 00:45:27.219
Gleb Otochkin: the API on backend, it is…

289
00:45:27.410 --> 00:45:33.510
Gleb Otochkin: On API, it is another whole conversational analytic agent, which

290
00:45:33.710 --> 00:45:43.349
Gleb Otochkin: connect to each other and understand, oh, I have intent from the customer, and I know what query template will serve for that. Also.

291
00:45:43.350 --> 00:45:54.569
Gleb Otochkin: to add to that query template, I add query facet, and the facet, it is a small piece of query, for example, some kind of conditions you want to use.

292
00:45:54.600 --> 00:46:02.790
Gleb Otochkin: And how it is written will depend whether, for example, index is going to be used or not. Function-based index.

293
00:46:02.840 --> 00:46:09.740
Gleb Otochkin: for example, will be used in Postgres only if the condition is written a special way.

294
00:46:09.810 --> 00:46:18.680
Gleb Otochkin: the same way as indexes created, right? And you have create query facets for that condition, and then you combine with query template.

295
00:46:18.700 --> 00:46:29.210
Gleb Otochkin: And then you generate the query, and execute query optionally, and get results back. That's how query data context set works. And you are…

296
00:46:29.220 --> 00:46:40.030
Gleb Otochkin: able to test it, of course, at Google Cloud, we have CodeLabs, how to use with query data context, and it is available, in CodeLab, I believe.

297
00:46:40.270 --> 00:46:43.380
Gleb Otochkin: It is kind of…

298
00:46:44.090 --> 00:46:56.450
Gleb Otochkin: you have two options. You can use it with MCP Toolbox, and you can use it with Google Remote Query Data Interface at Google Cloud. So, there are two different ways.

299
00:46:59.760 --> 00:47:08.779
Gleb Otochkin: So, that is recorded DMI. I don't want to use recorded DMI. Before going… hold on, let me, go…

300
00:47:09.490 --> 00:47:12.050
Gleb Otochkin: Before going forward.

301
00:47:12.330 --> 00:47:22.520
Gleb Otochkin: Let's make sense what we have. For example, you have anti-gravity or any kind of CLI, AI…

302
00:47:22.750 --> 00:47:23.760
Gleb Otochkin: ED?

303
00:47:24.850 --> 00:47:36.589
Gleb Otochkin: you can use it with managed MCP Server as a, for example, database management person, like DBA, DevOps, or anybody else. Or,

304
00:47:36.700 --> 00:47:39.200
Gleb Otochkin: conversational analytics.

305
00:47:39.570 --> 00:47:44.449
Gleb Otochkin: With MCP Toolbox, it is more suitable for local development.

306
00:47:44.550 --> 00:47:46.969
Gleb Otochkin: And self-managed databases.

307
00:47:49.430 --> 00:48:05.469
Gleb Otochkin: when you use anti-gravity, not CLI, but anti-gravity, it is probably the best for wipe coding. And MCP Toolbox, it is vibecoding with your local database. And of course, you also have AI Studio, it is

308
00:48:05.660 --> 00:48:23.219
Gleb Otochkin: I think a great thing about AI Studio. Now, AI Studio is integrated with some database engines as well, like Firestore and Cloud SQL as well. So, that's kind of trying to make sense from all that zoo of the tools and MCP servers available for you.

309
00:48:24.760 --> 00:48:42.260
Gleb Otochkin: We have, of course, at Google, we created, some MCP resources for you, if you want to scan what, what it is there. It is, on GitHub, at Google MCP, and CodeLab, I mentioned already, for… about NL,

310
00:48:42.260 --> 00:48:44.810
Gleb Otochkin: to SQL using query data. So.

311
00:48:44.810 --> 00:48:48.799
Gleb Otochkin: Try it, try it out, let us know, and be…

312
00:48:49.040 --> 00:48:54.410
Gleb Otochkin: bef- and now, before I'm going to the demo, I would like to…

313
00:48:54.510 --> 00:49:00.410
Gleb Otochkin: stop, and if you have any questions, I'm happy to answer. Or we can do it after the demo as well.

314
00:49:06.760 --> 00:49:08.579
Gabor Szabo: I think you can do the demo.

315
00:49:08.870 --> 00:49:12.830
Gabor Szabo: And, a few people have questions, Yeah.

316
00:49:13.240 --> 00:49:15.549
Gleb Otochkin: Okay, let me go to the demo.

317
00:49:17.550 --> 00:49:22.079
Gleb Otochkin: So, speaking about the demo, what I have here, I have a box

318
00:49:23.020 --> 00:49:30.150
Gleb Otochkin: On that box, I have a toolbox, MCP toolbox, deployed here, and I have

319
00:49:30.290 --> 00:49:38.289
Gleb Otochkin: tools, it is parameter file for MCP toolbox, it is tools.yaml. Let me make it a little bit bigger, probably, right?

320
00:49:39.120 --> 00:49:40.050
Gleb Otochkin: Alright.

321
00:49:41.040 --> 00:49:42.400
Gleb Otochkin: Is it…

322
00:49:42.400 --> 00:49:42.970
Gabor Szabo: Yeah.

323
00:49:42.970 --> 00:49:44.090
Gleb Otochkin: good enough?

324
00:49:44.090 --> 00:49:48.620
Gabor Szabo: Maybe a little bit larger, I mean, I have a big screen, but

325
00:49:49.520 --> 00:49:52.840
Gleb Otochkin: Okay, fuck yeah.

326
00:49:53.820 --> 00:49:56.760
Gabor Szabo: And just… just the font's a little bit larger.

327
00:49:56.760 --> 00:49:57.470
Gleb Otochkin: Okay, it is…

328
00:49:57.470 --> 00:49:58.240
Gabor Szabo: Both of them.

329
00:49:58.240 --> 00:49:59.280
Gleb Otochkin: I'm trying to deliver.

330
00:49:59.280 --> 00:50:00.100
Gabor Szabo: No, please.

331
00:50:00.970 --> 00:50:01.500
Gleb Otochkin: Yup.

332
00:50:01.870 --> 00:50:02.700
Gleb Otochkin: Okay.

333
00:50:12.770 --> 00:50:16.599
Gleb Otochkin: why it is disconnected from other sessions, I don't know.

334
00:50:19.890 --> 00:50:22.900
Gabor Szabo: The younger people in the audience say that they can see finally.

335
00:50:25.110 --> 00:50:29.750
Gleb Otochkin: Oh, I don't know why it is disconnecting. I don't like it.

336
00:50:31.970 --> 00:50:32.910
Gleb Otochkin: Okay.

337
00:50:34.540 --> 00:50:35.580
Gleb Otochkin: But, yeah.

338
00:50:41.460 --> 00:50:42.720
Gleb Otochkin: Alright…

339
00:50:42.720 --> 00:50:45.420
Gabor Szabo: Yeah, it's much better, I don't know, yeah.

340
00:50:45.620 --> 00:50:51.380
Gleb Otochkin: Okay, let me… just to… it is 3 different windows here,

341
00:50:51.690 --> 00:50:58.910
Gleb Otochkin: 3 different windows, and those windows will be using… Now, what is going on?

342
00:51:00.530 --> 00:51:01.930
Gleb Otochkin: Yeah, okay.

343
00:51:02.330 --> 00:51:08.790
Gleb Otochkin: So… What I have here, let me clear it here, okay…

344
00:51:09.850 --> 00:51:14.090
Gleb Otochkin: I have, deployed some simple agent code.

345
00:51:14.090 --> 00:51:18.850
Gabor Szabo: I just want to say that it's a nice touch that you use Database Maven as the… Several.

346
00:51:19.810 --> 00:51:24.010
Gleb Otochkin: I created that environment for that particular presentation, yes.

347
00:51:24.010 --> 00:51:25.319
Gabor Szabo: Nice. So…

348
00:51:25.650 --> 00:51:37.180
Gleb Otochkin: So, we have here the agent, agent using, ADK deployment, and, it is relatively simple,

349
00:51:39.640 --> 00:51:44.800
Gleb Otochkin: Python script with, Relatively simple quote here.

350
00:51:44.970 --> 00:51:56.650
Gleb Otochkin: it is using minimum, all the resources and requests and everything else, so it is very, very simplified. So, it has one tool.

351
00:51:56.840 --> 00:52:01.490
Gleb Otochkin: And that tool is connecting to MCP Toolbox.

352
00:52:01.720 --> 00:52:10.049
Gleb Otochkin: deployed locally, and it is one of functions also I implemented. It is how much tokens we spend, right?

353
00:52:10.210 --> 00:52:22.319
Gleb Otochkin: That's the, agent code. And it is very, very simple. It is created based on, just a couple of commands from ADK CLI.

354
00:52:25.180 --> 00:52:38.649
Gleb Otochkin: That's… the agent. Also, what we have here, we have tools.yaml for the toolbox, and if we get…

355
00:52:41.840 --> 00:52:44.140
Gleb Otochkin: tools.yaml.

356
00:52:44.420 --> 00:52:46.330
Gleb Otochkin: That is,

357
00:52:47.100 --> 00:53:01.519
Gleb Otochkin: configuration file for MCP Toolbox, and it provides the information about the resource, database, user password, and everything else. Of course, it is in clear text, but it doesn't mean it has to be in clear text. The MCP Toolbox

358
00:53:01.760 --> 00:53:18.119
Gleb Otochkin: has very advanced authentication and everything else, so that can be done. And what I'm here providing the tools, around the user's journey. For example, the… one of the first tools, search for hotels based on name.

359
00:53:18.180 --> 00:53:30.439
Gleb Otochkin: Then, in order to search hotels by location. For example, let's say we have some kind of travel system behind the scenes, and we are trying to work with that travel system.

360
00:53:30.660 --> 00:53:41.330
Gleb Otochkin: And, based on that, also you can book hotels or update, information about hotels, right? That's kind of tools we're providing.

361
00:53:41.640 --> 00:53:45.090
Gleb Otochkin: And, of course, we have that toolbox itself.

362
00:53:45.290 --> 00:53:52.240
Gleb Otochkin: And we have a Postgres database, and… And if we go to…

363
00:53:58.430 --> 00:54:00.580
Gleb Otochkin: Local host…

364
00:54:09.450 --> 00:54:16.510
Gleb Otochkin: I believe it is a database called ToolboxDB. Okay…

365
00:54:18.050 --> 00:54:22.950
Gleb Otochkin: If we go the, database itself… What is it?

366
00:54:24.390 --> 00:54:26.429
Gleb Otochkin: I don't see the error.

367
00:54:31.480 --> 00:54:32.630
Gleb Otochkin: Two books…

368
00:54:36.170 --> 00:54:37.680
Gleb Otochkin: Is it the right one?

369
00:54:39.800 --> 00:54:42.660
Gleb Otochkin: Toolbox, user, password, my password.

370
00:54:44.380 --> 00:54:47.949
Gleb Otochkin: Oh, yeah, it is wrong here.

371
00:54:50.650 --> 00:54:53.140
Gleb Otochkin: No, it is something toilet.

372
00:54:54.360 --> 00:54:56.420
Gleb Otochkin: connection failed.

373
00:54:56.720 --> 00:54:58.990
Gabor Szabo: He doesn't like the word localhost there.

374
00:54:59.390 --> 00:55:06.670
Gleb Otochkin: Yeah, I don't know why. Let me go and… Make a little bit cheat.

375
00:55:10.660 --> 00:55:12.080
Gleb Otochkin: Oh, hold on.

376
00:55:18.300 --> 00:55:19.630
Gleb Otochkin: Alright…

377
00:55:23.120 --> 00:55:28.789
Gleb Otochkin: And here we have database, we have 2BoxDB, let's connect to 2BoxDB.

378
00:55:29.980 --> 00:55:41.020
Gleb Otochkin: And here we have some tables, and we have tables with hotels, right? And if we select… from…

379
00:55:42.770 --> 00:55:55.649
Gleb Otochkin: we can see some different information, whether it is booked, what kind of photos we have, we can check in checkout dates and everything else when they're available. So, it is very, very simple schema, right?

380
00:55:55.950 --> 00:56:16.000
Gleb Otochkin: So, if we… execute… Alright… toolbox itself, with, our config tools YAML?

381
00:56:16.260 --> 00:56:24.010
Gleb Otochkin: Here we execute, and… For example, we can now access that MCP server

382
00:56:24.140 --> 00:56:37.210
Gleb Otochkin: for example, by just using quarrel command and getting to the backend. It is how every single MCP server is supposed to work, because if you put a post

383
00:56:37.380 --> 00:56:39.279
Gleb Otochkin: For example, right?

384
00:56:39.580 --> 00:56:41.580
Gleb Otochkin: To that one. Where is it?

385
00:56:42.570 --> 00:56:45.829
Gleb Otochkin: Let me open the demo requests.

386
00:56:48.340 --> 00:56:57.800
Gleb Otochkin: Okay… If we put, for example, Simple Coral, and we…

387
00:56:58.230 --> 00:57:05.119
Gleb Otochkin: Can we list meet tools by name? And we just filter, by one output.

388
00:57:06.280 --> 00:57:18.529
Gleb Otochkin: it will provide all our tools from configuration files, what we defined, right? That is how it's supposed to work, and you technically can get more information if you just

389
00:57:18.690 --> 00:57:35.390
Gleb Otochkin: get a little bit deeper insight, and you can always search hotel by name, it is description, it is what it does, it is the name of the hotel, and everything else. So, that information is… should be available. It is how MCP server works. You put the request, you get the result.

390
00:57:35.930 --> 00:57:40.449
Gleb Otochkin: So… You also search…

391
00:57:41.560 --> 00:57:49.129
Gleb Otochkin: you can do a little bit more advanced and everything else. So, that works really well, right? But…

392
00:57:49.520 --> 00:57:53.560
Gleb Otochkin: It is for our tools, but it doesn't mean we,

393
00:57:53.840 --> 00:58:09.410
Gleb Otochkin: have to use the same way. Do you remember about the pre-built tools? When we don't know what kind of user journey is going to be, we don't know what kind of tools we want to create. It is more suitable for development itself. So, in that case.

394
00:58:10.390 --> 00:58:17.450
Gleb Otochkin: What we can do… We can provide the…

395
00:58:18.600 --> 00:58:24.400
Gleb Otochkin: toolbox with parameter. I don't know if you can see here, it is toolbox with…

396
00:58:25.410 --> 00:58:30.520
Gleb Otochkin: pre-built Postgres tools. So, it requires some extra,

397
00:58:31.570 --> 00:58:44.070
Gleb Otochkin: environment variables to work with it, but essentially, it is the same, like, Postgres host, password, user, and everything else. So, when you start with pre-built tools.

398
00:58:44.350 --> 00:58:59.729
Gleb Otochkin: and go back and try to list the tools, what you have, tools by name, you are getting completely different set of tools. It is default tools for Postgres database. It is list stored procedures.

399
00:58:59.770 --> 00:59:16.950
Gleb Otochkin: stats, available extensions, and everything else, and you can execute query, you can get query plan, for example, and so on. So those pre-built tools are when you don't really know what kind of critical user journey you have, but you want to use that MCP server.

400
00:59:17.000 --> 00:59:21.660
Gleb Otochkin: Right? And so, it is how it is working behind the scenes. So…

401
00:59:22.110 --> 00:59:25.130
Gleb Otochkin: In that case, your agent can choose

402
00:59:25.680 --> 00:59:33.949
Gleb Otochkin: from pre-built tools and go through complicated, journey, for example, right? So…

403
00:59:34.450 --> 00:59:41.850
Gleb Otochkin: Let's… what we can try to do right now, we can… oh, white is jumping?

404
00:59:42.400 --> 00:59:45.620
Gleb Otochkin: Let's start the agent interface here.

405
00:59:46.720 --> 00:59:51.939
Gleb Otochkin: And the agent interface, it is, again, ADK,

406
00:59:52.060 --> 01:00:05.270
Gleb Otochkin: I'm starting, using UV Run ADK web, it is Python, developer. And of course, ADK has different languages. ADK, it is, Agent Development Framework from Google.

407
01:00:05.390 --> 01:00:14.419
Gleb Otochkin: So, I'm starting the web interface. What it allows me to, connect, using the web interface

408
01:00:15.320 --> 01:00:19.109
Gleb Otochkin: Let me try to and see if it works here.

409
01:00:19.260 --> 01:00:21.080
Gleb Otochkin: Yeah, it works here.

410
01:00:22.700 --> 01:00:27.890
Gleb Otochkin: So, we have a new session,

411
01:00:29.290 --> 01:00:48.650
Gleb Otochkin: And what we can do, remember, we have pre-built tools, we don't have predefined user journey. The agent, by itself, have no idea what exactly behind the database. What agent has… agent has the basic prompt, you are database assistants.

412
01:00:49.020 --> 01:00:51.439
Gleb Otochkin: For example, we put hello here.

413
01:00:51.970 --> 01:00:57.300
Gleb Otochkin: And agent know, oh, I have toolbox toolset.

414
01:00:58.160 --> 01:01:07.750
Gleb Otochkin: And I am connecting to Toolbox Toolset, and understand what I can do. And here, what it is responding. Let me make it a little bit bigger.

415
01:01:10.330 --> 01:01:13.280
Gleb Otochkin: It… it is responding, hello.

416
01:01:13.590 --> 01:01:23.010
Gleb Otochkin: how can I assist you with your database today? Feel free to ask about table inspection. So, it doesn't really know what it can do. It is universal.

417
01:01:23.170 --> 01:01:24.710
Gleb Otochkin: What I can ask.

418
01:01:27.890 --> 01:01:34.869
Gleb Otochkin: Can, you find a hotel in Basel?

419
01:01:37.670 --> 01:01:46.200
Gleb Otochkin: And let's see what it is going to do. So, since it is null, it knows I can work with Postgres database.

420
01:01:46.350 --> 01:01:54.050
Gleb Otochkin: And it is what it is doing behind the scenes. It is smart enough to list tables, understand

421
01:01:54.350 --> 01:01:57.639
Gleb Otochkin: Then, understand the table structure.

422
01:01:57.750 --> 01:02:02.010
Gleb Otochkin: and then execute SQL statement and provide me information.

423
01:02:03.200 --> 01:02:06.990
Gleb Otochkin: Only concern for me, it is how many steps

424
01:02:07.240 --> 01:02:16.649
Gleb Otochkin: Requires to provide that information, and if you can see here, total tokens, it is 36,000 tokens, right?

425
01:02:16.840 --> 01:02:18.170
Gleb Otochkin: So, that's…

426
01:02:19.320 --> 01:02:29.209
Gleb Otochkin: not big, but it is still kind of significant number of steps, so we spent 12 steps. Just to be fair, if I ask…

427
01:02:30.050 --> 01:02:32.990
Gleb Otochkin: What about hotel?

428
01:02:33.680 --> 01:02:36.840
Gleb Otochkin: in… Montreal.

429
01:02:39.250 --> 01:02:46.610
Gleb Otochkin: It will spend much less steps, it is only two steps, and essentially you have only 50,000 tokens.

430
01:02:46.940 --> 01:02:49.570
Gleb Otochkin: Two questions, which is not big, but still.

431
01:02:49.710 --> 01:02:55.779
Gleb Otochkin: We are working with very simplified database schema.

432
01:02:55.960 --> 01:03:13.129
Gleb Otochkin: it would be way more questions and back and forth if your schema has 15 different tables, and the information is spread around different tables, and it has to make a connection from one table to another, and another, and another. So, that's the idea. So.

433
01:03:14.540 --> 01:03:27.350
Gleb Otochkin: What if we do the same, but… we… start our… toolbox with our pre-configured.

434
01:03:27.530 --> 01:03:36.640
Gleb Otochkin: tools around user journey, right? Let's do that. We're starting our toolbox on MCP server, It is,

435
01:03:36.910 --> 01:03:50.569
Gleb Otochkin: with tools YAML, we are restarting our agent. Remember, we had 50,000,581, tokens so far?

436
01:03:50.860 --> 01:03:54.700
Gleb Otochkin: Let's restart our agent from scratch.

437
01:03:57.310 --> 01:04:04.970
Gleb Otochkin: put here, new session, and do exactly the same, questions. We are asking, hello.

438
01:04:05.610 --> 01:04:09.650
Gleb Otochkin: And see what it can work with.

439
01:04:11.370 --> 01:04:21.010
Gleb Otochkin: And you can see that even first response, the general hello, is already different, because it analyzed the tools and possessions.

440
01:04:21.240 --> 01:04:34.220
Gleb Otochkin: from the MCP, and oh, I can work with hotels here, and now I'm responding differently. I can work with hotel database, feel free to ask.

441
01:04:34.450 --> 01:04:42.000
Gleb Otochkin: Can you find me a hotel in Basel?

442
01:04:44.270 --> 01:04:46.929
Gleb Otochkin: And I'm asking the same question.

443
01:04:48.790 --> 01:04:53.480
Gleb Otochkin: And them getting, less… steps.

444
01:04:53.830 --> 01:04:57.419
Gleb Otochkin: I'm getting predefined, response.

445
01:04:57.900 --> 01:05:02.010
Gleb Otochkin: And… You can compare the number of tokens.

446
01:05:02.700 --> 01:05:14.370
Gleb Otochkin: This 8,000… plus 400 tokens, ex… instead of 36,000, right? Let's… What about…

447
01:05:24.320 --> 01:05:31.900
Gleb Otochkin: And the second question, it is, again, two steps. I'm getting, the information, what I want, and…

448
01:05:32.680 --> 01:05:36.199
Gleb Otochkin: The number of tokens is only 13,000.

449
01:05:37.020 --> 01:05:42.989
Gleb Otochkin: Versus, you probably remember, we had… Where is it?

450
01:05:44.130 --> 01:05:46.400
Gleb Otochkin: 50,000 tokens before.

451
01:05:46.720 --> 01:05:56.799
Gleb Otochkin: That's the difference between the tools created for user journey, specified agent for MCP, and tools created just…

452
01:05:57.370 --> 01:06:12.350
Gleb Otochkin: general purpose, and let an agent to understand how to work it out with all the metadata and everything else behind the scenes. That's my demo, and now probably I should switch back to the questions.

453
01:06:13.860 --> 01:06:21.940
Gleb Otochkin: I'm stop sharing right now, and… Let's go back to that.

454
01:06:24.170 --> 01:06:27.990
Gabor Szabo: Your, your screen is… your camera is off.

455
01:06:29.770 --> 01:06:37.659
Gleb Otochkin: aw… Okay, let me… let me find… Where is my interface?

456
01:06:39.160 --> 01:06:40.770
Gleb Otochkin: Boy, I can't…

457
01:06:40.770 --> 01:06:42.060
Gabor Szabo: It disappears.

458
01:06:43.750 --> 01:06:44.590
Gleb Otochkin: Oh.

459
01:06:46.410 --> 01:06:51.339
Gleb Otochkin: I, I, I, I'm trying to find my interface. Oh, yeah, I know where it is, yeah.

460
01:06:51.880 --> 01:07:00.000
Gleb Otochkin: Okay, I'm back. So… Any questions so far, guys?

461
01:07:00.400 --> 01:07:06.519
Gleb Otochkin: I'm happy to answer and discuss anything what you've seen so far, heard so far.

462
01:07:06.630 --> 01:07:08.149
Gleb Otochkin: What do you think about it?

463
01:07:10.580 --> 01:07:12.580
Gabor Szabo: Yeah, you… could you… oh, okay.

464
01:07:16.040 --> 01:07:19.089
Gabor Szabo: Sorry, now you can unmute yourself. I turned it off.

465
01:07:19.610 --> 01:07:27.360
Alejandro Imass: I think it was very clear. There are some things I already knew about some of these things that you explained. Obviously, you're focusing it to the Google…

466
01:07:27.930 --> 01:07:34.049
Alejandro Imass: Cloud, obviously, they… The object of the presentation,

467
01:07:34.800 --> 01:07:45.939
Alejandro Imass: But yeah, it clarified some context. I didn't know about the stateless MCP, so I learned quite a few things. Very interesting, presentation, very clear. No questions for me, it was very, very clear.

468
01:07:46.280 --> 01:07:49.049
Alejandro Imass: Very well presented, so I really liked it.

469
01:07:49.210 --> 01:07:50.460
Alejandro Imass: I think it was worthwhile.

470
01:07:51.250 --> 01:07:51.910
Gleb Otochkin: Thanks.

471
01:07:55.330 --> 01:08:01.979
Gabor Szabo: If anyone else wants to still ask questions here during the video, that's, that's fine, or say anything.

472
01:08:02.360 --> 01:08:06.499
Gabor Szabo: If not, then, then we'll, we, we'll…

473
01:08:06.960 --> 01:08:13.790
Gabor Szabo: And the video, and then those people who are here can stay on, and we can.

474
01:08:13.790 --> 01:08:20.019
Gleb Otochkin: I see some questions… I see some questions in the notes from Emmanuel,

475
01:08:20.740 --> 01:08:33.889
Gleb Otochkin: How do you control and monitor token usage when the large database schemas or query results are injected into LM context? That's a great question.

476
01:08:34.649 --> 01:08:39.600
Gleb Otochkin: you… So, the first couple of things.

477
01:08:39.750 --> 01:08:41.840
Gleb Otochkin: The token's usage

478
01:08:42.439 --> 01:08:53.249
Gleb Otochkin: depends on the context. Contents, sorry. So, it is, I had a presentation, probably, year or two…

479
01:08:53.359 --> 01:08:57.170
Gleb Otochkin: ago, how the different type of content

480
01:08:57.620 --> 01:09:04.220
Gleb Otochkin: imparts that token usage. For example, with,

481
01:09:05.910 --> 01:09:20.529
Gleb Otochkin: Content for, some large language models, you can have very big contents window, you can put entire book, War and Peace, from Leotal's story to that contents.

482
01:09:20.529 --> 01:09:27.809
Gleb Otochkin: But on the same time, you cannot put even small database table in the same contents.

483
01:09:28.000 --> 01:09:36.240
Gleb Otochkin: which is 100 or thousands times smaller in size. Why is that? Because

484
01:09:37.740 --> 01:09:41.329
Gleb Otochkin: Talking is not a symbol of worth.

485
01:09:42.109 --> 01:09:48.319
Gleb Otochkin: And token can be different. When you put the text in the tokens.

486
01:09:48.870 --> 01:09:55.949
Gleb Otochkin: Then the tokens will be recognized by either one word or part of the word.

487
01:09:56.360 --> 01:10:05.019
Gleb Otochkin: And that is great. But when you have, for example, flight table, for your United flights.

488
01:10:05.150 --> 01:10:06.820
Gleb Otochkin: with numbers.

489
01:10:07.290 --> 01:10:10.119
Gleb Otochkin: Then, each number will be talking.

490
01:10:11.460 --> 01:10:15.019
Gleb Otochkin: It means each number, and each…

491
01:10:15.800 --> 01:10:26.280
Gleb Otochkin: point, or anything else, we'll be talking as well. So, in that case, it is first thing. So, it is why you want to, limit how much,

492
01:10:26.520 --> 01:10:33.579
Gleb Otochkin: Information, how much Information you want to get out of the database.

493
01:10:33.720 --> 01:10:36.840
Gleb Otochkin: and provide to your OM context window.

494
01:10:37.540 --> 01:10:45.820
Gleb Otochkin: And you need to remember that not every content can be cached the same way on LLM site.

495
01:10:46.060 --> 01:10:53.220
Gleb Otochkin: And in your agent as well, so… If you can't cash content, on…

496
01:10:53.920 --> 01:11:00.339
Gleb Otochkin: Llm side, that's great. If your agent framework allows you to do that, that's…

497
01:11:00.680 --> 01:11:16.990
Gleb Otochkin: good for you. If you can't, then you have to slim down any… for example, during retrieval augmented generation, you don't want to retrieve whole table, you want to retrieve only few rows to provide contents. So, that's how you control it.

498
01:11:17.660 --> 01:11:20.710
Gleb Otochkin: Is it easy to do? Not really, it is hard.

499
01:11:21.210 --> 01:11:34.059
Gleb Otochkin: you know what you're… what you're working with. You have to know your data, you have to know your schema, you have to know how… what kind of tools to provide your MCP server to minimize that interaction with your data.

500
01:11:35.390 --> 01:11:41.530
Alejandro Imass: I have a… I have… I actually had a question. And it's regarding context. When you mentioned it, it reminded me…

501
01:11:43.190 --> 01:11:44.530
Alejandro Imass: I guess,

502
01:11:45.040 --> 01:11:53.590
Alejandro Imass: all LLMs, and I'm guessing all agents, will suffer from context lag as the context grows, the conversation grows over time.

503
01:11:53.860 --> 01:11:56.749
Alejandro Imass: The models get exponentially slower.

504
01:11:56.890 --> 01:12:01.319
Alejandro Imass: Like, inversely, I just want to show this lower, and pretty quickly. It doesn't take much.

505
01:12:01.920 --> 01:12:06.530
Alejandro Imass: To saturate the… so you have to reboot these agents, very frequently and clear their…

506
01:12:06.870 --> 01:12:08.789
Alejandro Imass: Contact start all over again, or…

507
01:12:09.220 --> 01:12:14.279
Alejandro Imass: What is the strategy there for the… I'm guessing the agents are the ones that are going to start lagging pretty quickly.

508
01:12:14.840 --> 01:12:17.649
Alejandro Imass: As they… as the context grows.

509
01:12:18.240 --> 01:12:22.119
Alejandro Imass: context of the conversation, a conversation we're having right now with the agent.

510
01:12:22.370 --> 01:12:26.549
Alejandro Imass: about trips. It doesn't take a lot to saturate,

511
01:12:27.190 --> 01:12:29.139
Alejandro Imass: And to start, you know,

512
01:12:30.910 --> 01:12:33.920
Alejandro Imass: Providing bad context to the conversation, and, you know.

513
01:12:34.030 --> 01:12:39.580
Alejandro Imass: hallucinations, stuff like that. So, what is the… the strategy there? Like, rebooting the agent, frequently? What…

514
01:12:39.690 --> 01:12:45.339
Alejandro Imass: I mean, you talked about scale at the very beginning, is one of the reasons, but… and one of the things that I have found

515
01:12:45.810 --> 01:12:51.450
Alejandro Imass: Is that it scaled horribly because of… and a growing conversation will,

516
01:12:51.600 --> 01:12:55.880
Alejandro Imass: and very quickly, saturate most elements out there.

517
01:12:56.890 --> 01:13:02.470
Gleb Otochkin: Yeah, good question. So, yes, longer you keep conversation with the agent.

518
01:13:03.430 --> 01:13:17.399
Gleb Otochkin: more context you have, like, historical context, all the interactions, and depending on the model, and of course, on the model side, in most modern… model as a service, it is going to be cached, right?

519
01:13:17.660 --> 01:13:19.990
Gleb Otochkin: So you don't really,

520
01:13:20.870 --> 01:13:29.810
Gleb Otochkin: do a lot of back and forth again, but still, it is getting slower and slower and slower, because even if it is cached, it has to be analyzed.

521
01:13:30.350 --> 01:13:32.490
Gleb Otochkin: For each single response.

522
01:13:32.810 --> 01:13:35.089
Alejandro Imass: And I found no easy way, there's no…

523
01:13:35.090 --> 01:13:36.279
Gleb Otochkin: So, the…

524
01:13:36.280 --> 01:13:38.740
Alejandro Imass: That way, to clear the context and start over, so it's…

525
01:13:38.760 --> 01:13:42.529
Gleb Otochkin: But, you, you can compare your contents.

526
01:13:42.800 --> 01:13:59.079
Gleb Otochkin: What I mean by other compress your contents, what you can do… do you remember we were talking about episodic memory for, on the database side? So what you can do, you can store all that long conversation on the database side, but from time to time.

527
01:13:59.200 --> 01:14:12.420
Gleb Otochkin: You… ask… Internally, your agent, create… when the context size reach certain size, create summary of that context.

528
01:14:13.140 --> 01:14:18.880
Gleb Otochkin: and replace the context. So you're creating short summary what was before.

529
01:14:19.370 --> 01:14:25.820
Gleb Otochkin: and discarding the main context body. So you make it small again, without losing,

530
01:14:25.990 --> 01:14:32.060
Gleb Otochkin: subject of conversation itself. That's… we call Campreya's context.

531
01:14:32.620 --> 01:14:37.210
Gleb Otochkin: Of course, if you don't… if you discard it and forget about that, that…

532
01:14:37.400 --> 01:14:40.279
Gleb Otochkin: Reduce your quantity of your responses.

533
01:14:41.570 --> 01:14:47.120
Gleb Otochkin: Because sometimes you have to retrieve that old memory, and that is where you want to

534
01:14:47.220 --> 01:14:53.119
Gleb Otochkin: Search inside your database and get the piece of chunks of context back to your conversation.

535
01:14:53.120 --> 01:15:04.230
Alejandro Imass: I see the… I see what you're proposing as a pattern is kind of keeping the growing context in the database, only breach it, or only access it if you need to, but try to work with the summary.

536
01:15:04.380 --> 01:15:06.030
Alejandro Imass: Yep. For most cases.

537
01:15:06.160 --> 01:15:10.340
Alejandro Imass: Okay, I got it, I got it. That's a… that's a nice pattern. Okay.

538
01:15:12.540 --> 01:15:13.430
Alejandro Imass: Wow.

539
01:15:13.430 --> 01:15:15.910
Gleb Otochkin: Another question, I believe, was…

540
01:15:20.290 --> 01:15:24.759
Alejandro Imass: It was interesting, because when Alberto told me about this workshop.

541
01:15:25.860 --> 01:15:29.839
Alejandro Imass: The idea of agents and databases didn't immediately click.

542
01:15:31.380 --> 01:15:37.030
Alejandro Imass: But now I see value in this, in this idea. Yeah.

543
01:15:37.030 --> 01:15:47.519
Gleb Otochkin: So, we actually have a lab, code lab, my colleague, at Google Engineering, created a lab how to create that

544
01:15:47.770 --> 01:16:03.980
Gleb Otochkin: upload the context. They created lab for AloIDB, but AlloyDB, essentially, Postgres behind the scenes, so you can technically get the lab, it is just sample code, what you can do, and they created the lab using not ADK, but blank chain graph.

545
01:16:04.100 --> 01:16:11.299
Gleb Otochkin: Land graph interface framework for agent, and what… what they do, they…

546
01:16:11.490 --> 01:16:16.730
Gleb Otochkin: Use the database to store in the context and everything else, and the episodic memory, and…

547
01:16:16.860 --> 01:16:29.579
Gleb Otochkin: If your agent somehow dropped again, and you can restart it and blow out memory back, so that's… they created Lab. I didn't try that lab by myself, it just released, probably, like, a week or two ago.

548
01:16:29.580 --> 01:16:32.550
Alejandro Imass: We… there was,

549
01:16:32.900 --> 01:16:38.900
Alejandro Imass: context marking the conversations would be nice as well. You can, like, mark certain context.

550
01:16:39.270 --> 01:16:42.730
Alejandro Imass: You know, like, at a certain point in the conversation, market.

551
01:16:43.030 --> 01:16:45.900
Alejandro Imass: And then relating that to an action or something that…

552
01:16:46.330 --> 01:16:51.279
Alejandro Imass: all the response that was obtained, that those techniques I've also seen are quite interesting.

553
01:16:51.610 --> 01:16:52.590
Alejandro Imass: Yeah.

554
01:16:52.590 --> 01:17:11.580
Gleb Otochkin: And technically, you can use… when you upload the context memory to the database, you create… split it to chunks, and then, create embedding letters on that sense, and then it will much… it will be much easier to find that pieces of context using vector search.

555
01:17:13.420 --> 01:17:19.279
Alejandro Imass: Very interesting. This is… Very enlightening. Thank you.

556
01:17:20.320 --> 01:17:20.950
Gleb Otochkin: Welcome.

557
01:17:22.100 --> 01:17:23.230
Gleb Otochkin: Anybody else?

558
01:17:26.030 --> 01:17:32.199
Emmanuel Thouraud: Yes, I have another one. How do you control the SQL queries generated by the MCP?

559
01:17:32.520 --> 01:17:35.840
Emmanuel Thouraud: To prevent a poorly constructive or recursive query.

560
01:17:36.390 --> 01:17:43.480
Emmanuel Thouraud: The goal is to avoid consuming excessive PostgreSQL resource or running indefinitely.

561
01:17:43.800 --> 01:17:53.970
Emmanuel Thouraud: Are we able to set some self-ground, such as query time hours, or limiting the resource, or even a kind of query validation on the MCP itself?

562
01:17:55.140 --> 01:18:01.370
Gleb Otochkin: So, it depends. It is kind of… so, MCP is a just toll call, right?

563
01:18:01.800 --> 01:18:08.369
Gleb Otochkin: So, when you execute a tool call, you can provide some guardrails.

564
01:18:08.490 --> 01:18:10.490
Gleb Otochkin: For example, for…

565
01:18:11.590 --> 01:18:21.939
Gleb Otochkin: For example, in Google Cloud MCP Server, Managed MCP Server, I'm not talking about the options for MCP Toolbox, it has its own options.

566
01:18:22.190 --> 01:18:28.679
Gleb Otochkin: How to limit, for example, execution query, how to make it read-only, how to print on some other stuff.

567
01:18:29.360 --> 01:18:39.259
Gleb Otochkin: You… either provide For example, for MCP Toolbox, I showed we have a tool

568
01:18:39.730 --> 01:18:42.970
Gleb Otochkin: For example, search hotel by city, right?

569
01:18:43.520 --> 01:18:53.910
Gleb Otochkin: In the example, and I provide the query inside the tool. That query will be executed. So, in that case, you know what kind of query is going to be executed, right?

570
01:18:54.100 --> 01:19:09.909
Gleb Otochkin: And in that case, you don't expect anything else. But if I provide only pre-built tool, you don't know what kind of query to be… will be executed. In that case, you have few options. The first option, you first

571
01:19:10.110 --> 01:19:17.500
Gleb Otochkin: You don't want to change… if you don't want to change anything in your database, you provide read-only access to your database, right?

572
01:19:17.800 --> 01:19:20.310
Gleb Otochkin: It can be done on different levels.

573
01:19:20.500 --> 01:19:22.749
Gleb Otochkin: It can be done on user level.

574
01:19:23.210 --> 01:19:39.670
Gleb Otochkin: what user can do and what user cannot do. For example, you can select from all tables for that particular schema, but you cannot change anything on those tables, right? So that's on database level. Also, what you can do

575
01:19:39.880 --> 01:19:43.920
Gleb Otochkin: You can provide Time out for the query, as well?

576
01:19:44.790 --> 01:19:51.330
Gleb Otochkin: You can tell, or if query is executed in more than 5 seconds, terminated.

577
01:19:52.510 --> 01:19:58.259
Gleb Otochkin: connection. That's… For example, for Google Cloud, manage…

578
01:19:58.600 --> 01:20:01.949
Gleb Otochkin: MCP servers, I believe it is 30 seconds.

579
01:20:02.570 --> 01:20:08.489
Gleb Otochkin: by default, If query is executing more than 30 seconds by default.

580
01:20:08.510 --> 01:20:24.390
Gleb Otochkin: then it will be terminated. It is why, for example, I believe I… I'm not sure if I provided the example why we're providing extra parameters. If you know your query will be executed more than 30 seconds, you can provide the parameters time out for the query.

581
01:20:24.430 --> 01:20:31.409
Gleb Otochkin: In that case, you know. For example, you have analytical query, you know that query is going to run 5 minutes.

582
01:20:31.490 --> 01:20:37.499
Gleb Otochkin: And you know you're gonna wait for that query, you don't want to terminate it in the middle, so…

583
01:20:37.710 --> 01:20:39.879
Gleb Otochkin: that option, but…

584
01:20:40.000 --> 01:20:50.419
Gleb Otochkin: It is… you have to technically work based on your data and your user journey, really. It is why I say that MCP Server is not just API.

585
01:20:50.880 --> 01:21:09.410
Gleb Otochkin: It is about user journey, what kind of requests, and query data is working really well in that sense. Of course, you don't have query data with open source Postgres, but at Google Cloud, if you use query data, in that case, it has internal mechanism preventing a lot of things behind the scenes to just

586
01:21:09.500 --> 01:21:11.060
Gleb Otochkin: plain safe.

587
01:21:11.330 --> 01:21:19.559
Gleb Otochkin: Some of them are documented, some of them are not documented, but in general, it is, like, if query is executed on query data.

588
01:21:19.860 --> 01:21:25.969
Gleb Otochkin: If it will be executed in read-only by default, unless you specify it explicitly.

589
01:21:27.840 --> 01:21:31.329
Gleb Otochkin: So, that's where you can kind of…

590
01:21:31.560 --> 01:21:44.559
Gleb Otochkin: be more or less on the safe side. But again, yes, there are a lot of uncertainty, and the models are unpredictable, as you know, right? And you want to reduce that,

591
01:21:44.900 --> 01:21:47.279
Gleb Otochkin: Uncertainty and,

592
01:21:48.010 --> 01:21:55.259
Gleb Otochkin: Weigh how the model works, providing the templates, facets, and everything else to make the query what you expect.

593
01:21:58.070 --> 01:22:06.310
Gleb Otochkin: Because, yeah, it can create absolutely ridiculous query with, tons of mergers and everything else, yeah.

594
01:22:08.800 --> 01:22:09.790
Emmanuel Thouraud: Okay, thank you.

595
01:22:11.260 --> 01:22:13.529
Gleb Otochkin: I hope I answered the question, at least I tried.

596
01:22:19.230 --> 01:22:22.649
Gleb Otochkin: Any other questions, guys? We still have 3 minutes, right?

597
01:22:28.300 --> 01:22:31.020
Emmanuel Thouraud: So, yeah, so one more I have in mind.

598
01:22:32.210 --> 01:22:40.769
Emmanuel Thouraud: I think, yeah, as you spent a lot of time on this MCP, did you benchmark the approach MCP, versus a non-MCP implementation?

599
01:22:41.400 --> 01:22:41.740
Gleb Otochkin: Yeah.

600
01:22:41.740 --> 01:22:44.600
Emmanuel Thouraud: To compare the quality on, okay.

601
01:22:44.600 --> 01:22:52.939
Gleb Otochkin: So… The fir- most of comparison lately is coming from, do we need MCPU or skills?

602
01:22:53.670 --> 01:22:54.490
Emmanuel Thouraud: Yeah.

603
01:22:55.090 --> 01:22:58.010
Gleb Otochkin: So… what I found…

604
01:22:59.140 --> 01:23:08.820
Gleb Otochkin: It depends, again, it depends. When you have MCP with predefined query, usually what you compare in that case, it is, like, pre-built tools for, or

605
01:23:09.230 --> 01:23:18.800
Gleb Otochkin: like… Widely scoped MCP tools, like, default MCP tools for Postgres, and set of skills, right?

606
01:23:19.130 --> 01:23:25.050
Gleb Otochkin: Because, technically, what you do When you execute skills.

607
01:23:25.820 --> 01:23:30.240
Gleb Otochkin: Skills, it is kind of reaction to agent what to do, right?

608
01:23:30.300 --> 01:23:49.919
Gleb Otochkin: In that case, you provide direction to the agent, okay, here are the tools you have for skills. If user asks you to create database, execute that SDK call, right? And you have SDK inside your agent, and skill, okay, I have that tool to execute that SDK code.

609
01:23:49.920 --> 01:23:58.899
Gleb Otochkin: call. It can be actually a REST API call to interface, right? And that is create. Mcp will work

610
01:23:59.660 --> 01:24:03.539
Gleb Otochkin: differently, because in that case, you communicate to MCP,

611
01:24:04.410 --> 01:24:11.620
Gleb Otochkin: with the tools and parameters for the tool, and MCP will execute tools. So, what I found is

612
01:24:12.990 --> 01:24:28.950
Gleb Otochkin: sometimes, depending on type of the request, skills can be better than MCP, sometimes MCP can be better than skills, it depends on the request, but I found if you really know what your users are going to do, you combine both together.

613
01:24:30.630 --> 01:24:40.599
Gleb Otochkin: In that case, for example, sometimes it is much easier to use one tool to create something which MCP will be using later.

614
01:24:40.600 --> 01:24:41.100
Emmanuel Thouraud: there.

615
01:24:43.510 --> 01:24:48.620
Gleb Otochkin: Skills are more universal than MCP, by default, as from, you know…

616
01:24:50.500 --> 01:24:52.830
Emmanuel Thouraud: Yeah, it's sometimes more simple to set up.

617
01:24:53.290 --> 01:25:09.240
Gleb Otochkin: Yes, it is, like, you don't need to do anything just to read SkillsMD file, and everything is alright, but again, when you work with databases, it can be kind of a little bit misleading, because you might get not results you really expect.

618
01:25:09.310 --> 01:25:18.860
Gleb Otochkin: It is working really well with different SDK, but the databases You still need data interface, Right?

619
01:25:18.960 --> 01:25:35.899
Gleb Otochkin: Or you have to connect directly to your database somehow, and sometimes it is not as easy as, and as protected. And another whole, absolutely different world, I'm going to speak in a month on PGConf in New York.

620
01:25:36.130 --> 01:25:38.280
Gleb Otochkin: It is security and MCP.

621
01:25:39.890 --> 01:25:51.109
Gleb Otochkin: nobody asked here, but it is a kind of big problem with security. It is a lot of patterns you want to prevent from prompt injection, SQL injection,

622
01:25:51.510 --> 01:25:59.129
Gleb Otochkin: I mean, drop in your database, right? If you put in the prompt, don't drop my database, it doesn't mean agent will not do that.

623
01:26:00.370 --> 01:26:01.529
Emmanuel Thouraud: Yeah, of course.

624
01:26:01.700 --> 01:26:02.230
Gleb Otochkin: that.

625
01:26:07.820 --> 01:26:17.179
Alejandro Imass: REST, as well, for access. There's other… not necessarily direct connection database, we also have PGREST, PulseREST, there's a bunch of, like, REST interfaces to…

626
01:26:17.770 --> 01:26:21.700
Alejandro Imass: Postgres, I run… In the database itself, and serve as a…

627
01:26:21.990 --> 01:26:23.930
Alejandro Imass: REST API to the outside world.

628
01:26:25.150 --> 01:26:38.119
Gleb Otochkin: Yeah, it is different, different layers. Also, the layer, for example, if you want to protect only piece of your data in your schema. And in that case, you probably want to

629
01:26:39.050 --> 01:26:45.429
Gleb Otochkin: Use some database, Mechanism to prevent reading from that table.

630
01:26:45.970 --> 01:26:53.729
Gleb Otochkin: And there are some different ways. We created private security views, for that. It means,

631
01:26:53.920 --> 01:27:06.510
Gleb Otochkin: In that case, only if you are probably… it is kind of role-level access rules, which helps to prevent some data. You want to see… you need to see only what you…

632
01:27:07.290 --> 01:27:13.250
Gleb Otochkin: allows… a lot to see, right? You don't want to provide everything to all users.

633
01:27:15.760 --> 01:27:17.760
Alejandro Imass: A lot of food for thought, definitely.

634
01:27:20.660 --> 01:27:23.230
Alejandro Imass: Oh, pretty cool. I have a hard stop right now, so I have to…

635
01:27:23.230 --> 01:27:23.750
Gabor Szabo: joke.

636
01:27:23.980 --> 01:27:28.969
Alejandro Imass: Thank you very much, guys. This is a very, clip, and, Gabor for putting this together.

637
01:27:29.500 --> 01:27:32.019
Alejandro Imass: It was really great. I really enjoyed it.

638
01:27:32.340 --> 01:27:32.869
Alejandro Imass: Thank you.

639
01:27:32.870 --> 01:27:33.510
Gleb Otochkin: Thank you.

640
01:27:34.170 --> 01:27:34.760
Gabor Szabo: Hmm.

641
01:27:35.160 --> 01:27:38.159
Gabor Szabo: Yeah, Gleb. Thank you very much for this presentation.

642
01:27:38.710 --> 01:27:39.460
Gabor Szabo: And thank you.

643
01:27:39.460 --> 01:27:39.950
Gleb Otochkin: You're welcome.

644
01:27:39.950 --> 01:27:48.380
Gabor Szabo: Everyone, all the people who were here and asked questions, or didn't ask questions, you're still welcome.

645
01:27:48.770 --> 01:27:49.170
Gleb Otochkin: Yeah.

646
01:27:49.170 --> 01:27:53.919
Gabor Szabo: And, people who are watching the video… I think we are finishing now, right?

647
01:27:54.540 --> 01:27:55.390
Gleb Otochkin: Yeah. Yep.

648
01:27:55.390 --> 01:28:07.530
Gabor Szabo: Okay, so, people who are watching the video, please like the video, and follow the channel, and remember that below the video, you will find links to,

649
01:28:08.500 --> 01:28:24.089
Gabor Szabo: thinks about this presentation, probably maybe also the slides, I'm not sure, we'll see. Definitely to access to where you can find, grab, and to the future events. So, thank you very much, and see you in,

650
01:28:24.220 --> 01:28:25.419
Gabor Szabo: The next session.

651
01:28:25.860 --> 01:28:26.659
Alejandro Imass: Thank you very much.

652
01:28:26.660 --> 01:28:27.520
Alberto Mijares: Thank you very much.

653
01:28:27.520 --> 01:28:28.130
Gleb Otochkin: Thanks.

654
01:28:28.130 --> 01:28:29.949
Alberto Mijares: It's great, right? Really great.

655
01:28:30.470 --> 01:28:31.660
Marlene: Thank you.

656
01:28:32.660 --> 01:28:33.610
Marlene: Thank you.

