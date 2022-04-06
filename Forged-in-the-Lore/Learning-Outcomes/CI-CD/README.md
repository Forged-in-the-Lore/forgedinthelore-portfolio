```mermaid
graph LR
subgraph External
  direction LR
  failed((Merge Failed))
  artifact[(Artifact Storage)]
  dockerrepo[(Docker Image Repository)]
  sonarcloud[[Sonar Cloud]]
  click sonarcloud https://sonarcloud.io/project/overview?id=OIBSS-F_Forged-In-The-Lore
  end
subgraph Backend
  bm([Merge]) --> bb1([Build])

  subgraph Unit Testing
    bb1--> sonarcloud
    bb1 --> bt1([Unit Test])
    bt1 --Test failed-->failed
    end

  subgraph Integration Testing
    bt1 --Tests Passed-->bt2([Run and Seed DB])
    bt2 --> bt3([Integration Test])
    bt3 --Tests Failed--> failed
    bt3 --Tests Passed--> bt4([Publish Backend Artifact])
    bt4 --> artifact
    end

  subgraph Delivery
    artifact --> bd1
    bt3 --Tests Passed--> bd1([Pull Website Artifact])
    bd1 -->bd2([Build Docker Image])
    bd2 -->bd3([Deliver Docker Image])
    bd3 --> dockerrepo
    end
  end

subgraph Frontend
  fm([Merge]) --> fb1([Build])

  subgraph Unit Testing
    fb1--> sonarcloud
    fb1 --> ft1([Unit Test])
    ft1 --Test failed-->failed
    end

  subgraph Frontend Testing
    ft1 --Tests Passed-->ft2([Run and Seed DB])
    ft2 --> ft3([Integration Test])
    ft3 --Tests Failed--> failed
    ft3 --Tests Passed--> ft4 
    end
  end
```
