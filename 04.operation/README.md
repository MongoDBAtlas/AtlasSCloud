<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Handson Workshop for Samsung SCloud

## Backup & Restore
데이터베이스 운영에서 중요한 부분인 백업과 복구 부분을 실습합니다.

### [&rarr; Snapshot Config](#Snapshot)

### [&rarr; Continuous Snapshot](#Continuous)

### [&rarr; Point in time restore](#Restore)


<br>


### Snapshot

Atlas의 백업은 EBS의 Snapshot 백업과 유사하게 작동 합니다. Scondary 노드 중에 하나가 백업 대상이 되어 스냅샷형태로 백업이 됩니다. 클러스터가 생성된 리전에 백업이 생성 됩니다.   
Snapshot 백업 설정을 위해 Atlas console에 로그인 합니다.  

생성한 클러스터를 클릭 하고 Backup을 클릭 합니다.   

<img src="/04.operation/images/image01.png" width="50%" height="50%">     

스냅샷의 정책을 확인 합니다. 해당 시간이 되어야 백업이 진행 되어 시간이 소요 됨으로 On-demand backup을 수행 하도록 합니다. Snapshot 을 클릭 하고 take snapshot now를 클릭 합니다.

<img src="/04.operation/images/image02.png" width="50%" height="50%">     

스냅샷에 대한 설명을 작성하고 보유 기간을 입력 후 snapshot을 클릭 합니다.

<img src="/04.operation/images/image03.png" width="50%" height="50%">     

스냅샷 생성은 몇분이 소요 됩니다.

몇분후 백업이 생성 된 것을 확인 할 수 있습니다.

<img src="/04.operation/images/image04.png" width="50%" height="50%">     



### Continuous
Continuous Backup 으로 특정 시간으로 복구하기 위한 설정을 합니다. 클러스터의 Edit Configuration을 클릭 하고 Continuous Backup을 On 하여 줍니다.

<img src="/04.operation/images/image05.png" width="50%" height="50%">     

클러스터 정보를 변경 하여 줍니다. (적용은 몇분이 소요 됩니다.)     
백업 설정을 위해 클러스터 정보의 Backup을 선택 하여 줍니다.

<img src="/04.operation/images/image06.png" width="50%" height="50%">     

Point in time restore policy의 Restore window 입력을 확인 합니다. 해당 시간은 Snapshot중 hourly snapshot의 보관 주기보다 작아야 합니다. 복구 과정에서 hourly snapshot이용하여 복구 후 restore window 기간 동안 보관한 로그에서 Snapshot 시간과 복구하려는 시간 동안의 데이터를 복구 하여 줍니다.
(설정 내용으로 현재 시간 기준 7일 이전의 시간으로 복구가 가능 합니다)

#### Restore

특정 시간으로 복구를 위해 데이터를 생성 하여 줍니다.  Mongosh을 이용하여 데이터를 2 차례 생성 하여 줍니다. (1분 차이를 두고 생성 하고 시간을 기록 하여 줍니다)   

````
Atlas atlas-wwrcm3-shard-0 [primary] test> use backup
switched to db backup

Atlas atlas-wwrcm3-shard-0 [primary] backup> for (i = 0; i < 10; i++) { let insertDoc = { number: i, create_at: new Date() }; db.backupcol.insertOne(insertDoc);}
{
  acknowledged: true,
  insertedId: ObjectId("6484109fc539c07478c60252")
}
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.find().count()
10
Atlas atlas-wwrcm3-shard-0 [primary] backup> 

````

10건의 데이터가 들어간 것을 확인 할 수 있으며 생성한 시간을 확인 해 봅니다.

````
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.findOne()
{
  _id: ObjectId("64841679e5fc659d04711bd6"),
  number: 0,
  create_at: ISODate("2023-06-10T06:21:45.232Z")
}
````

1분 이상 지난 후 다시 데이터를 생성 하여 줍니다.

````
Atlas atlas-wwrcm3-shard-0 [primary] backup> for (i = 10; i < 20; i++) { let insertDoc = { number: i, create_at: new Date() }; db.backupcol.insertOne(insertDoc);}
{
  acknowledged: true,
  insertedId: ObjectId("648416a7e5fc659d04711be9")
}
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.find().count()
20
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.findOne({number:19})
{
  _id: ObjectId("648416a7e5fc659d04711be9"),
  number: 19,
  create_at: ISODate("2023-06-10T06:22:31.660Z")
}
````

전체 데이터의 개수가 20 개 인것을 확인 할 수 있습니다. 시간이 ISODate("2023-06-10T06:22:31.660Z")으로 1분의 차이가 있는 것을 확인 할 수 있습니다.    

데이터 복구 테스트를 위해 클러스터 내부에 Backup을 클릭 하고 Point in time restore를 클릭 합니다.   


<img src="/04.operation/images/image07.png" width="50%" height="50%">     


시간을 첫번째 10개 데이터가 생성된 시간을 선택 합니다. ISODate("2023-06-10T06:21:45.232Z")

<img src="/04.operation/images/image08.png" width="50%" height="50%">     

다음을 선택 하여 복구 대상 클러스터를 선택 합니다. 테스트를 위한 것임으로 현재 클러스터를 선택 합니다. (현재 데이터를 무시하고 지정한 시간의 데이터로 복구 됩니다.)   

<img src="/04.operation/images/image09.png" width="50%" height="50%">     

복구에 수분이 소요 됩니다.

복구 완료 후 클러스터 컬렉션의 정보를 확인 하면 데이터가 10건이 있는 것을 확인 할 수 있습니다.

<img src="/04.operation/images/image10.png" width="50%" height="50%">     

Mongosh 로 데이터를 확인해 보면 전에 생성한 10개의 데이터 만 있는 것을 확인 할 수 있습니다.  

````
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.find({number:9})
[
  {
    _id: ObjectId("64841679e5fc659d04711bdf"),
    number: 9,
    create_at: ISODate("2023-06-10T06:21:45.322Z")
  }
]
Atlas atlas-wwrcm3-shard-0 [primary] backup> db.backupcol.find({number:10})

Atlas atlas-wwrcm3-shard-0 [primary] backup>

````