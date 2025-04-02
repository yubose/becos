# becos
Blockchain Empowered Content Organizing System



 BECOS(Blockchain Empowered Content Organizing System) SDK Design Specification

Hongtao Yu
©August 3, 2007 - March 30, 2025
General Description	2
Acronyms	2
TIME : ctime(8), mtime(8), atime(8), atimes(4), tage(4)	3
Vid, Eid and Did are unique id of objects (particle) of ecos system.	3
vTable: vertex table, node table	4
eTable: edge table, or relation table.	5
dTable: -- document table	6
Doc Object Revision Control	7
SignIn, SignUp, Password Reset and Account Forfeit and Reclaim	9
Helper Functions	13
SDK Functions	13
CvReq : {"jwt":"string", "Vertex":"{Vertex Object}"}	14
CeReq, CdReq having the same structure. the only difference is the carried Object are 'Edge' and 'Doc' respectively.	14
RxReq: {	14
DxReq : {	22
Error Code and Message	26
Push Notification	29
To maintain the participant list in a group chat	33
To get invited group list	33
To get documents in a list	34
SignIn/SignUp Logic	35
Document Types	35
DocType:	35
1: PubProfile:    "256" #              0x100	35
1: Profile:          "257"                  0x101  private,	35
2: Contact:       "513"                   0x201, private	35
2: ContactFav:  "515"                  0x203 private | favorite	35
GetAllContact: "type in (513,515)"	35
GetFavContact: "type=515"	35
4: UploadFile:   "1025"                  0x401  private updated document	35
5: InviteUserMessage: "1281"      0x501  private invitation	35
6: MeetingNote: "1537" # 6<<8+1 6	35
Notification Processing Queue	38
Database cache and update Queue	39

General Description
	
Information is created and consumed by human beings. Information is the relations among many entities which are "named" by human beings. An 'Event" is an action taken by a human being at a specific time for a specific purpose. The emission of an "Event" is the same as the atom emitting an "electron." The result, consequence of an action, will yield a relation in the information ledger. Therefore, the 'e' of eCOS stands for "edge" and its two aspects of "entry" of the relation as an object and "Event" of action. "Event" can trigger another "Event" to happen. Every "Event" is signed and chained as a rooted tree.

Server preserves the consistency and security of the data and the Edge defines the performance.

	Life's time component: the cycle of "Life" is described by its "Time" properties. For each information node, its time properties are recorded and documented.w

Acronyms 

PK  : public key
SK  : secret key
ESK : symmetrically encrypted secret key
SAK : symmetric access key
ESAK : asymmetrically encrypted symmetric access key.
SPW : scrambled password

VID : Vertix ID
EID : Edge ID
bVID : begin vertex on the directive graph.
eVID : end vertex on the directive graph.

DID : Document ID


TIME(32) : ctime(8), mtime(8), atime(8), atimes(4), tage(4) CAMAT cama-T
ctime : create time, ctime is never changed.
mtime : modify time
atime : last access time
atimes : access times, counter for the number of access.
tage : target age
Time derived attributes:
Boolean isNewborn() = ctime == mtime
	isDead() = atimes < 0; abs(atimes) is the age of dying. -1 is dead after a newborn.
		Object will be moved TOMB db
	atime is initiated as 0; Once the content is completed and ready for use, atime is set to mtime;
	isMature() = atime > 0;
                isRetired() = tage < 0; Object will be moved to History DB for revision access
Moving objects to TOMB or Revision DB will be done separately from main thread.

Use TIME as stop-watch to record time elapsed: et.ResetTimer() will set tage to 0
Each time, et.TimeIt() call, Now()-MTime() in second will accumulated on tage.
Use TIME as Timer, SetTimer(nsecond) will set to tage. And IsExpired will check if et.Age() > et.Tage.
Age() is the Now()-Ctime in second.
ATimeReached() = et.Atimes == 0
Access() will increase Atimes and update Atime accordingly.
AccessTimes or Atimes could be set to a negative value, once Access() increases it to Zero, ATimeReached() will be true.


Vid, Eid and Did are unique id of objects (particles) of ecos system.

Vid, Eid and Did oid is based on UUID Universal Unique identifier. It is a 16 byte of 8 bits. total 128 bits.

https://en.wikipedia.org/wiki/Universally_unique_identifier

360a 8ebc 2963 4bdb 2ced ed76 c810 6f61
ex:
00110110  00001010 10001110 10111100
 00101001 01100011 00001011 11011011 red mark the version numb is 0, ECOS generate
 00101100 11101101 11101101  01110110
 11001000 00010000 01101111 01100001

According to the UID standard, there are up to 16 different version, and only 1 to 5 is defined and used. And the 3 most significant bits of the 9th byte are used to represent variant


First two bits of the 9th byte to indicate the ID type is VID=00, EID=10, DID=01, FID=11
byte 0 to byte 3 is the integer giving the low 32 bits of the time
byte 4 to byte 5 is integer giving the middle 16 bits of the time
The 4-high-bits of byte 6 is the version number defined by UUID stands. And Version=1 to 5 have been used.
ECOS-SERVER will use VERSION=0 to define ECOS IDs
Version=0(ECOS) is similar to Version=1(date-time and MAC address)

The difference are
a) ECOS will use UNIXTIME in 256 microsecond incremental,
b) reserve the high four bits of byte 9 as the "variants"
c) This leaves ECOS has total 72 bits for timestamps.

HIGHTIME byte6[5:]-byte7-byte4-byte5-byte0-byte1-byte2-byte3 are the create-time in second
LOWTIME: byte8[5:]-byte9 are the least significant of the creating clock.

unixtime.now().microsecond>>8 i.e divided by 256 is the time stored in the ID

The least significant 12 bits are stored in the LOWTIME,
Then the most significant 60 bits are stored in the HIGH TIME fields.

Therefore, from ID, its create time can be reconstructed. 

This gives ECOS
the lower 4 bits of byte 6 and byte 7 is the high 12 bits of the time.

The resolution of time is 256 micro-second. The rollover year is 36,558,903,054AD.

d) node id is the MAC address at which machine the DB is born. This node id stays with the DB. 
    If a ecos server's node ID is confirmed by APPLE-Relation.



LIVE(80): TIME(32), Name(4), Deat(4), Type(4), Subtype(4), ID(16)  
id: uuid -- binary(16)
name:text varchar(64k) json searchable public description, no encrypted confidential information should be stored here.
dear:text varchar(64k),  is the attribute derived from its edges. It is a JSON<k,v> string derived from edges.
	There one edge created, may need updated its two vertice.



VERTEX(216): LIVE, pk(32), esk(72), uid(32), 
vTable: vertex table, node table
The attributes of a vertex are defined by the relation of its edges.
uid : varchar(32) external user identifiable id, such as a phone number or email address.
pk: public key -- binary(32)
esk: encrypted secret key -- binary(64)
v.isIdentifiable() == uid != null;
Vp is a vertex with a verifiable uid, such as a phone number or email.
Vo is a vertex created by Vp as an organization, such as a company, a family or a clinic.

There are two kinds of Vertex, one is generated by the system for newly registered users. The other is created by a user(registered) to create a new relation. "Relation" generated "Vertex".

Cost of the change of v.key.
re-encrypt all SAK on all 'edges'.

Cost of the change of e.SAK, need to re-encrypt all files which were encrypted by old SAK and update ESAKs of all 'edges'.

An admin of an organization is authorized to access the v.key of the Organization.
The bVid of the authorization 'edge' is the owner

doas : "do as" 

EDGE(288): LIVE(80), bVid(16), eVid(16), sTime(8), eTime(8), refid(16),
bESAK(56), eESAK(56), sig(32)
eTable: edge table, or relation table.

sig is signed by bVid.SK for the fields of etype + TIME.ctime + TIME.mtime + bVid + bESAK + eVid +  eESAK.
bESSK: docid access symmetric 
bvid : who creates this relation, sign the signature of the authorization.
	who owns the right to the fate of the documents vault. who can authorize eVid to view and modify the files in the vault.
eVid : whom this relation is target to. This is the authorized viewer.
name : text varchar(64k), searchable public description of this relation
subtype: specific subtype derived from the eType.
refid : reference to other entries on the eTable.


eid acts as the folder or vault id

Edge:represents the relationship between two vertice.
The attributes of edge defined the semantic function of the 'edge'.

e.isPublic() = e.bESAK == null
e.isPrivate() = e.bESAK != null && b.eESAK == null
e.isKEYA() = e.bESAK != null && b.eESAK != null // is accessible with a KEY
e.isRoot() = e.refid == 0;

The 'sig' on a 'edge' can only be signed by bVid for the authorization of the access of the 'edge'.
?? Could 'edge' be re-signed ??
?? Could we consider that an un-signed 'edge' is not "matured" or ready for use ??

E-Type:
Apply: // require to start
Invite: // a vertex to start a relation
Tell: //message
Meet:
Open

Doc(144):LIVE(80), fid(16),  eid(16), size(4), ovid(16), reid(16)

dTable: -- document table

bSig(32) as primary key, since id could be duplicated by reference existing documents.
eSig(32) is provided after the eVid modified the file.
When both signatures are presented, this document is a contract.
For the docs with the same name, and dtype and eid, their timestamps track the revision history of the doc.

total bytes = 144
id : uuid document object id
dtype: int document type
name : text, varchar(64k) public searchable text field.
eid : edge id.
fid : fid could be overloaded with 'did', 'eid' or 'vid', which make the 'doc' object linked to
       edge, vertex and other doc.
bSig: the vid of the person who made the last changes of the Doc. The correct name for this field should be cid, i.e. creator's id.
eSig: the root-edge id of the Doc belonging to. If eid->Edge->refid == nil. eSig is nil. The correct name for this field should be rid, i.e. root-edge-id

Before a document is signed by the eVid, bVid can change the file without creating a new copy.
At this stage, the document is considered as in-complete or on-going creation. Its createTime is updated and new bSig is created for each update. Original createTime may be written into the name field.
After a document is signed by the eVid with TIME.mtime, this document cannot be updated further.
Any updates will create a new document which inherited the dtype, name, and eid, and a new id(did) will be created.
The previous file will be considered as historical revision.
	
	For the same d.name and d.dType and d.eid, there is only one copy of the file, additional copies are considered as previously revised version.

Permission of Creating/Updating a Doc Object 

1) Doc.eid cannot be nil
2) The user(JWT) of the action(creating or updating) a Doc has to be either the bvid or evid of the Edge which is pointed by Doc.eid.
3) When a Doc is updated by a user which is different from the creator, a force revision will be created.
Doc Object Revision Control
	a) Forced Revision Creation: when update a Doc Object, its 'fid' is set to its 'id', ECOS will take a snapshot of the current object in db and create a new Doc Object with a new 'id', and the existing Doc Object will be updated accordingly. The field of 'Fid' in db will not be changed. This fid=id is only used for creation of revision entry. Revised Doc Object will be moved to "backup/history" DB.
           b) AI revision creation, upon Doc Object updating, in following three conditions
1)Revision-age is defined as the time elapsed since last revision created in second.
 if the Revision-age is greater and 65535 seconds(more than 18 hours)
2) name change-factor is defined as 
 len(diff(new_doc.name, db_doc.name)) * (len(new_doc.name) + len(db_doc.name) is greater than 65535
            3) name-change-factor * revision-age > 65535
In plain language, if the name is changed after more than 18 hours, any minor change is trigger to revision creation. If a large file has minor changed in a short time, a revision is created as well. If a small change in a small file within a small time elapsed, no revision will be created.

[2020.10.22 2:21 PM] doc.type to doc.subtype migration

Major ECOS backend update:
1) add subtype to Vertex and Doc.
2) protorepo @ http://gitlab.aitmed.com/backend/protorepo.git has been updated as well.
3) For exist data compatibility, doc.type has copy to doc.subtype
4) For existing frontend sdk, there is no subtype suppied, ECOS will create duplicated type to subtype for future data compatibility.
5) Frontend SDK should use doc.subtype as current doc.type logic. Subtype belongs application logic. Therefore,
    complicated sCondition is allowed to do Rx queries.
6) Noodl is able to use doc.type with much fexibility.
SDK migration required:
1) change current doc.type to doc.subtype.
2) Pass doc.type from Noodl to ECOS

SignIn, SignUp, Password Reset and Account Forfeit and Reclaim

ECOS requires a user to sign up with a valid phone(mobile) number, which could receive a text message. Whenever a user(client) requires to retrieve encrypted credential from ECOS, ECOS will send a verification code of limited lifespan (currently 5 minutes) to the phone. The user needs to provide the verification code to retrieve his/her encrypted credential. This encrypted credential is only decrypted with the user's master password at the client-side. On trusted devices, the user is able to lock his/her local credential to prevent any unauthorized usage. The user has the choice to clean local credential. After the local credential is removed, the user is required to use a new verification code to retrieve his/her credential from ECOS.

1) Once a user is successfully identified by his/her own master password, the user is able to update/change the password by providing an existing password and new double-input-checked password. ECOS will update the encrypted credential for future usage.
only update esk, pk is needed as well. Server will check if it has been changed. If it has been changed, the update request will be rejected and need to use vc-JWT as next scenario. 

Technically, 'CV' command is used to update the ''TYPE"-1 Vertex, with valid vid-JWT

INVALID request, in this case, more identity check is required to claim the digital right of the existing account. GOTO case No. 4
 2) If the user forget his/her master password completely, He can recreate his/her credential with a new master password. In this case, valid verification-code is required. Because old credential is lost, all privately owned notebooks will not be able to access by any body. It won't be accessible since the master password is forgotten. But the secret-shared notebooks's access-right could be re-authorized by the user with whom the secret-shared notebooks have been shared with.

 Technically, 'CV' command is used to update the "TYPE"-1 Vertex with valid vc-JWT and Tage field is set to the user received verification-code.

3) Update phone number, requires valid verification-code and vc-vid-JWT. If the server found the existing usage of the phone number, ECOS_CODE(1030, Uid Has Been Used), When this happened, the user will be advised the cost of reclaimed the content associated with the existing phone number. If the user confirmed to claim the phone number, Vertex update API "CV" will used and "fFoOrRcCeE+new phone number" should be used to force ECOS to make the changes and the existing account which is associated the phone-number will be forfeited.

If a user holds valid local credential, even his/her phone-number on the 'uid' is forfeited, the user is still able to use this capability to enact a new phone number on this account. 
 
4) When a new user is created with a valid phone number and verification code, ECOS may find that the phone number has been registered. The user will be prompted that he is really want to re-create a new account with the phone number. And the user is advised that there will be a significant financial cost to reclaim the contents for the 'forfeited" account. After the user confirm his ownership with the phone number, "CV" creates a new Vertex with "TYPE"=-1 to force ECOS to disable the account associated with the phone number and create a new "TYPE"=1 account for this phone number. And the "TYPE" of the existing Vertex will changed to "-1" to express the forced forfeit. 
In this case, "VTYPE" is used as a command.
Implemented on 12/9/2019

5) User sets a Username. For "TYPE"=1 Vertex, 'uid' could be in username+countrycode phone-number format. The "username" is required to be unique cross ECOS. And the "CountryCode PhoneNumber" is required to be unique as well. "Username"  can be used to retrieve the encrypted credential from ECOS. Of course, valid verification code is required.
The 'username' could be changed with only valid vid-JWT, and ECOS_CODE(1030, Uid Has Been Used) will be returned if existing usage is found.
Limitation: length(uid) = 32
maximum length(phone number) = 16
uuuu uuuu uuuu uuuu+cccsppp ppp ppp pp
16 characters for user name
11 digitals for phone number
1 "+" sign in between
1 " " space in between the country code and phone number.
maximum length for country code is 3 digital. 



SendVerification-code API("ETYPE"=1010) could companion without JWT agree with bvid of the edge.

SendVerification-code API("ETYPE"=1011) could companion with a valid JWT agree with bvid of the edge.








Root-Notebook, Edge.type == 10000

          It is a personal file repository. It is not allowed to be shared with anybody. Doc created on other (type of) Edges could be backuped to the Root-Notebook. And for creating a certain type of Doc, the latest Doc of the same Doc.type.applicationType could be retrieved from the Root-Notebook. And any changes made on it could be prompted to the user, and give him the chance to replace the copy in Root-Notebook for future use.

For example, the Profile of a user was initially created in the Root-Notebook. Then, when the user makes a medical appointment with a doctor, the Profile is inherited from the Root-Notebook, and let the user make any changes. If there are any changes made, the application should prompt the user to see if the user wants to update the master-copy of the Profile in the Root-Notebook for future use. This logic could be apply to other types of Doc with different applicationType.

The Root-Notebook acts as personal knowledge (information templates) repository which could be inherited for creating(spawning) other Doc. 

Notebook for group sharing, Edge.type == 10001

Inbox, Edge.type == 10002

      Inbox is a one-to-one relation between two Vertex. Inbox is a Edge.

Re(Retrieve Edge) for all Inboxes
xfname = 'bvid|evid"
type = 10002

d(Retrieve Mails) of all inboxes.
xfname = 'eid'
id = id list of all inboxes
application data type =R 7
maxcount = 200



GroupChat, Edge.type == 10003

	This is similar to Note

ChatLine, Edge.type == 10004  like instance message


Its Edge.type == 10002
After user registered,created a  Vertex V0 with 3 edges(represented as 3 notebooks).
One with t = 10000, called RootNotebook, owned as personal one, won’t be shared with others.
One with t = 10001,will be the group notebook sharing with others, everyone who has been invited can make comments on it, using SQL query to select the Doc.id, and shows the contribution was made by whom.
The third one with t = 10002, called inbox.
If someone(e.x Vertex V1) has sent user V0 an invitation, there will be a link in the inbox after sign in, with the doc.eid, t = 10002, bvid = V1,evid = V0, refid != nil. 
And after confirmation, the other users(e.x Vertex V1) will also receive the message at their inbox with t = 10002, bvid = V0, evid = V1, refid != nil.
So, with the dual connections, User V0 and V1 can make friends with each other, their name will 
show on the friend list.





Helper Functions
API Functions to Server
.requestVerifySignUpPhone ({phone_number: str}) return void
Send the phone number to backend,  so the backend will send verification text message to user.

Encryption Functions
	blob == uint8array
.generateAKey () return : {"publicKey":blob, "secretKey":blob };
Generates a keyPair for asymmetric encryption/decryption.
.generateSKey() return blob
Generates a secretKey for symmetric encryption/decryption
.aKeyCheck(publicKey: blob, secretKey:blob) return true|false
Checks if the asymmetric keyPair is a valid one
.aKeyDecrypt(secretKey: blob, data:blob) return encrypted blob
Asymmetrically decrypts the given data using a secretKey from a valid keyPair
.aKeyEncrypt(publicKey: blob, edata:blob) return a decrypted blob
Encrypts the asymmetrically encrypted data using the publicKey from a valid keyPair
.sKeyEncrypt(secretKey: blob, data:blob) return encrypted blob
Symmetrically encrypts data using a secretKey
.sKeyDecrypt(secretKey: blob, edata:blob) return decrypted blob
Decrypts the symmetrically encrypted data using the secretKey it was encrypted with
.uint8ArrayToBase64(data:blob) return base64 string
.base64ToUint8Array(data:str) return blob
.uTF8ToUint8Array(data:str) return blob
.uint8ArrayToUTF8(data:blob) return string
S3
.uploadFileToS3 ({encrypted_file: blob}) return encrypted_file_S3_url : str
Upload the encrypted_file to the S3 and return the encrypted_file_S3_url
SDK Functions

	In this system, three level SDKs are defined.

 SDK-1: is generated by gRPC. It provides CRUD(Create, Retrieve, Update, Delete) of 'e'(Edge) 'd'(Doc) and 'v'(Vertex) from ECOS index repository -- backend servers. The ECOS index repository governs the behaviors of the various different of 'e', 'v', and 'd'. The differences are denoted by their types, the "type" is on every object.

1. Every request should provide a JWT(JSON Web Token), if the JWT is valid JWT at client side, JWT="" should be provided.

2. Every response from the server, a JWT will be returned. At the client side, the newly returned  JWT should be used in the subsequent request.

3. Each Request needs to submit with a JWT, even it is an empty string "" and a Request Object associated with the request.

4. Each Response will contain a JWT, an integer Code to indicate the status of the response and a string typed message Error to provide more interpretation of the status.
 
At the core level, ECOS only provides 7 APIs to cover the CRUD for all the three core objects V.E.D

CV : Create Vertex. It is also overloaded with Update capability when an valid "id" is provided in the submission objects.
CvReq : {"jwt":"string", "Vertex":"{Vertex Object}"}
CvResp: {"jwt":"string","code":"ddddd","Vertex":"{Vertex Object}"}

CeReq, CdReq having the same structure. the only difference is the carried Object are 'Edge' and 'Doc' respectively.

RxReq: {
           ObjType: int32
	ids : []IDList // if not presented, the JWT encrypted UUID is used instead.
                  All the ids in the 'id' list are UUID which are used with xfname to create query conditions.
  xfname  : the "field name", for which the "id" is compared against, default field name is "id".
Also, if xfname is "none", xfname condition will not be created.
                     xfname supports full-boolean operators as "!", "&", "|" for logic "NOT", "AND", "OR", and parentheses ")", "(" can be used to group the operators priorities of the boolean operation. 

<xfname> ::= <LTERM> <LOPT> <LTERM>
<LTERM> ::= '(' xfname ')' | <VARLIST>
<LOPT> ::= '&' | '|' | '!' 
<VARLIST> ::= <VAR> | <VARLIST> ',' <VAR>|<empty>
<VAR> ::= <singleVar> | <rangeVar>
<singleVar> ::= fieldName starts with lowercase letters.
<rangeVar> ::= fieldName starts with Uppercase letters.

<VARLIST> will be matched against ids in the IDList, If there is no match id, the var will be set to "IS NULL". <empty> <VAR> is a placeholder to let the next var to match the following id.



Examples:
      ids = [  1, 2 ] 

      1) xfname =  bvid | evid      will convert to
      (bvid = 1 OR evid=1)

       2) xfname = Bvid | Evid
       (bvid in (1,2) OR evid in (1,2)) // this is the case of using xfname as RangeVar

       3) xfname = (bvid | evid)&(,refid)
       (bvid = 1 OR evid = 1) AND (refid = 2)

       4) xfname = (bvid,evid) & (,refid)
       (bvid = 1 AND evid = 2) AND refid = 2

       5) xfname = !(bvid,evid) & (,refid)
       NOT(bvid=1 AND evid=2) AND refid=2

       6) xfname = !(Bvid|Evid) & (,refid)
       NOT(bvid in (1,2) OR evid in (1,2)) AND refid=2
 
  
      type   : the "type" of the objects are expected. If omitted, all types of objects are returned.
       key   : Optional, if omitted, no string search will be done.
                  Key is a string of search keys in boolean form such as
(key1 & key2 | key3 | key4) & !key5 to search against xfname
if key ends with "!", exactly matched key is required.
each key could be also in regular expression form. 
     sfname : the "field name", for which the 'key' is searched against, default field name is "name"
     // Pagination control
    loid      : Optional, if presented, its Object Type should be matched with the RxReq. It is the 'id' of the last item from previous returned RxReq.
maxcount : The maximum return item count, the default is 40, maximum allowance should be limited to 1000. This is about minimum 250KB payload on average. 
    obfname : Order by field name. is comma-separated field names.If omitted, 'mtime' is assumed.
  scondition : additional SQL condition string.
  asc            : a boolean variable. default is false, it means returns are in descending order.
}
is used for all the three different kinds of objects.
For Rx response, in addition to the common three field, {jwt, code, error}, there is always a list of JSON objects. The length of the list could be '0' to tell that there is no objects found.

ECOS Special Query Logic

ecos Special Query Logic(eSQL) defined by 'ObjType' 
bit0, bit1 for table status,
 0: live
 1:retired table(use for revision control),
 2:remove item, such trash can
 3: query on Edge(App local).

higher bits for special query logic.
eSQLType = ObjType>>2 & 07FFF
eSQLDBID = OjbType>>17
,
Then, ECOS has 32k different DB and 32k different special query logic for each of the 12 tables. 

special Query Logic is defined for each kind of the three base data objects(V,E,D)

eSQL

1) All Docs a user can see by Doc join to a group Edge 
eSQLType = 1 (ObjType=4)
SELECT D.* FROM Doc D INNER Edge E ON D.reid=E.refid
WHERE (jwt=E.bvid OR jwt=E.evid)
 AND E.type in (10000,40000);
api:rd
default:
Xfname = "E.bvid|E.evid"
sample: sCondition = E.type in (10000,40000)


SELECT DISTINCT X.* from XTable INNER JOIN Edge E on 

SELECT DISTINCT E2.* from Edge E2 INNER JOIN Edge E2 ON E.refid=E2.id"
SELECT DISTINCT D.* from Doc D INNER JOIN Edge E ON E.id=D.eid"
SELECT DISTINCT V.* from Vertex V INNER JOIN Edge E ON V.id=E.bvid OR V.id=E.evid"

2) Doc to Group Edge join query logic
eSQLType = 1 (ObjType=4)
SELECT D.* FROM Doc D
INNER JOIN Edge E on E.refid=D.reid
WHERE E.Refid='ids'

ids = ['doc.reid'] or ['edge.refid']
Xfname = "E.Refid" 
sCondition = "Etype=40000"

3) All my edges
This is a regular query
Xfname = id=refid AND bvid|evid

4) Vertex join to a group of Edge
api:rv
id: rootRoom.id
Xfname="E.refid"
ids = [edge.Refid]
sample: all participant
sCondition: "E.Type=40000 AND E.Refid IS NOT NULL AND V.Type=1"

5) example get all meeting histories:
api:re
id: JWT
xfname = "Ebvid|E.evid"
sCondition: "E2.Type=40000 AND E.Subtype=0 AND E.Type=1053 AND E.Tage>0 AND E2.Tage>0 E2.Refid=NULL"


6) ObjType encoding scheme
// bit0, bit1 for table status 0: live, 1: delete, 2: archived
// bit2, bit3, bit4 for legacy usage
// bit5, bit6 for joining table type, 0=vertex, 1=doc, 2=edge;
// Therefore Doc to Doc join query bit5 = 1, bit6=0
// bit8-bit31 are special query type. 8/25/2021

Type.bit0: private
Type.bit1: favorite
Type.bit2: read
Type.bit3: deleted

Doc to Doc join query
ObjType=288 or 0x120, It returns the D2.* and D.fid as fid  as the main query structure.
xfname=D.fid

select D2.*, D1.fid from Doc D inner join Doc D2 on D.reid=D2.id


ObjType=544 or 0x220, 
This is "myTag" join query,

It returns 
 "D.type&0xFF|D2.type as type"
 "D.subtype&0x1FFFF|D2.subtype as subtype"
It bit-ORs the lower 8 bits for D.type to D2.type as type to return
It also bit-ORs the lower 17 bits for D.subtype to D2.subtype as subtype to return
 "INNER JOIN Edge E ON D.eid=E.id LEFT JOIN Doc D2 ON D2.reid=D.id AND D2.ovid=userID and D2.type&0xFFFFFF00=0xE00"
default xfname is (E.bvid|E.evid)

 example:
{
  "jwt": "puVsL1SwTYRWTXkqimFEWQ==",
  "ObjType": 544,
  "type": 1281 // this the inbox message docType
}

ObjType=800 or 0x320
The D2 join condition is the same as ObjType=544

Action to ObjType=544, 800 is triple join query as
INNER JOIN Edge E on D.eid=E.id
INNER JOIN Doc D3 on D.id=D3.reid
LEFT JOIN Doc D2 ON D2.reid=D.id AND D2.ovid=userID and D2.type&0xFFFFFF00=0xE00"

This is used to return the two attributes related join query, such as return the emails under a folder(label) and reading and favorite status
default xfname is (E.bvid|E.evid)&(,D3.fid)
Examples:
ObjType=288 for BTC macAddress to  DeviceInfo query
{​
"id": [
"JPZ3E7wYAADAAAAAAAAAAA==" // encode MAC Address
],
"jwt": "QbyeNHpuQp54A2gZAjNm+A==", // could be anybody's jwt.
"ObjType": 288,
"type": 3072 // 0xB00 Blue Tooth DeviceInfo docType
}​
default Xfname is D.fid, join condition is D.reid=D2.id, return D2


ObjType=544 for inbox query with readTag and/or starTag

{​
  "id": [
    "" // default is jwt derived vid
  ],
  "jwt": "bD01LgxMQfBiIcNSvBU6Xg==",
  "ObjType": 544,
  "type": 1281, // inbox messageType
  "sCondition": " E.Type=10002"
// 3584 is MailReadTag docType
// 10002 is inbox edgeType
}​
default Xfname is E.bvid|E.evid
For inbox, which contains message from other people, Xfname should be like "(E.bvid|E.evid} & (!D.ovid), Not created by me
For folder of "sent", Xfname should be like "(E.bvid|E.evid)&(D.ovid) , for created by me.
join Condition is Doc D inner join Edge E on D.eid=E.id left join Doc D2 on D.id=D2.reid

ObjType=800 inbox query under a folder with readTag and/or starTag
currentUser.Vertex.ID is required
{​
  "id": [
    "MvV8GyjLRJo3rOkBLb2x1Q==", // This is currentUser.vertex.id
    "EyjsAoD1Qwd9NIrJgjU6+A==" // This is folder docId
  ],
  "jwt": "a2wgQkSgTHFjaNN3AB9Orw==",
  "ObjType": 800,
  "type": 1281
}​
default Xfname is (E.bvid|E.evid)(,D3.fid)
join Condition is Doc D inner join Edge E on D.eid=E.id inner join Doc D3 on D.id=D3.reid left join Doc D2 on D.id=D2.reid


mail1 id EsOwAWcXRBBMtInb1jZ2pA==
{
  "jwt": "O1b/MkoDQDBJZgBPfeyHQg==",
  "doc": {
    "type": 1281,
    "eid": "+g6DzHZ3Q9iIME03aRJXAA==",
    "subtype": 1,
    "name": "inbox message"
  }
}
notage attached.

mail id siFOTWOXSX1A+DLCsdV8Fw==
folder1 id EyjsAoD1Qwd9NIrJgjU6+A==
{
  "jwt": "O1b/MkoDQDBJZgBPfeyHQg==",
  "doc": {
    "type": 3328,
    "eid": "+g6DzHZ3Q9iIME03aRJXAA==",
    "subtype": 1,
    "name": "folder1"
  }
}
folderTagId tQvMzRixTpJ0NJB2oqhYsQ==
{
  "jwt": "O1b/MkoDQDBJZgBPfeyHQg==",
  "doc": {
    "type": 3840,
    "eid": "+g6DzHZ3Q9iIME03aRJXAA==",
    "subtype": 1,
    "name": "folderTag",
    "esig": "siFOTWOXSX1A+DLCsdV8Fw=="
  }
}
readTag: S+xjHR57RFVcWdaFc+jlQw==
{
  "jwt": "O1b/MkoDQDBJZgBPfeyHQg==",
  "doc": {
    "type": 3584,
    "eid": "+g6DzHZ3Q9iIME03aRJXAA==",
    "subtype": 65,
    "name": "readTag",
    "esig": "siFOTWOXSX1A+DLCsdV8Fw=="
  }
}
Return inbox message with readFlag, non-deleted message
{
  "id": [
    "" // default is jwt derived vid
  ],
  "jwt": "bD01LgxMQfBiIcNSvBU6Xg==",
  "ObjType": 544,
  "type": 1281,
  "sCondition": "(D2.Type&0xFFFFFF08=0xD00 OR D2.reid IS NULL)  AND E.Type=10002"
}

Return inbox message with readFlag, deleted message
{
  "id": [
    "" // default is jwt derived vid
  ],
  "jwt": "bD01LgxMQfBiIcNSvBU6Xg==",
  "ObjType": 544,
  "type": 1281,
  "sCondition": "D2.Type&0xFFFFFF08=0xD08  AND E.Type=10002"
}

Return inbox message with readFlag under a folder, not trashed.
{
  "id": [
    "MvV8GyjLRJo3rOkBLb2x1Q==", // This is currentUser.vertex.id
    "EyjsAoD1Qwd9NIrJgjU6+A==" // This is folder id
  ],
  "jwt": "a2wgQkSgTHFjaNN3AB9Orw==",
  "xfname": "(E.bvid|E.evid}&(,D3.fid)"
  "ObjType": 800,
  "type": 1281,
  "sCondition": "(D2.Type&0xFFFFFF08=0xD00 OR D2.reid IS NULL)  AND E.Type=10002"
}

Return inbox message with readFlag under a folder of trashed
{
  "id": [
    "MvV8GyjLRJo3rOkBLb2x1Q==", // This is currentUser.vertex.id
    "EyjsAoD1Qwd9NIrJgjU6+A==" // This is folder id
  ],
  "jwt": "a2wgQkSgTHFjaNN3AB9Orw==",
  "xfname": "(E.bvid|E.evid}&(,D3.fid)"
  "ObjType": 800,
  "type": 1281,
  "sCondition": "(D2.Type&0xFFFFFF08=0xD08 OR D2.reid IS NULL)  AND E.Type=10002"
}

For Edge bit5=0 and bit6=1
ESqlEdgeJoinEdge ObjType=1088
JWT is a userID
ID is vid of another user.
This query returns the root edge object(id=refid), in the edge group(grouped by refid),
both query_user(JWT userID) and ID of a participant user id.
This is triple join query.
DxReq : {
	jwt, []id
}
Object remove policies:
1) For 'Doc' Object, 'Fid' field has to be empty before deletion. And JWT has to be matched with the 'Eid" related 'Edge'.
2) For 'Edge' Object, no 'Doc' should be attached to it. Also, it should not be referenced by any other 'Edge'. The JWT should be matched with 'Edge'
3) For 'Vertex', it should not own any 'Edge' Object.

SDK-2: Manage the V, E, D objects, retrieve from the repository.

	Basic Object, v(Node, Vertex), e(Edge, Relation), d(Document)


.reqPin (user_id) request server to send a pin to the location associated by the user_id
return = {"err":"none" | "mismatch uid/pwd", "epin":"xxxxxxxxxxxx","jwt":"xxxxxx"}
epin is server side encrypted pin with the sk of the server.,
client decrypting the epin to match the user pin input.
once client side verified the pin. send it back with the jwt for server side verification to create the user account.

.CreateUser (user_id, name, password, pin), to confirm access by v(Node) , this creates the root user -- vertex 
 return = {"err":"none"|"user_id exists" | "pin mismatch"}
JS will generate pk and sk with user_id, password, time_stamp
JS will encrypt sk with the user's password.
call sign_api with

.CreateUser(jwt:str, name) //create child node.

.invite(jwt: str, user_id: str) // wait for the user_id to confirm, to authorize a specific access.
Organization can invite a patient, a doctor.
Doctor can invite an organization to join become associate, such as credentialing
User create an organization, AiTmed to approve its business identify.

.deactivate(jwt)
.login(user_id, password)
return = {"err_code":"nn", "err_msg":"msg", "jwt":"xxxxxx"}
check if local storage has a_key_pair
.logout()
clear local storage.
notify: server about logout.


SDK-3:Map application objects to VED objects.
Create v-to-v relation, edge




What is an organization?
O : Organization is a group of people who are calibrated to accomplish tasks or projects.
O : Organization has staff who works for the organization.
O : Organization has members or clients, who have authorized the organization to access certain files.
O : has a pair of keys(PK, ESK), ESK is initially encrypted by the organization's creator's ESK.
O : is able to the new ESK to different staff to access the ESK.
O : edge authorized to be accessed by 
O : authorize a member's access by using the member's PK to encrypt the O.SK.
O : can create organization, such as function department.‰
O : Vo->O relation and Oo->O relation are represented by edge.
O :  

Vertex Types:
External identifiable vertex with login credential, such as by phone number and password or by email and password, etc.
This is called IV, (identifiable vertex)
Every other vertex is created by an IV.
Dependent:
Member:

Sign Up
Use Case: Client A sign up


Upload File
Basic concept of Files and File Folders
Files
Each file must belong to a file folder
File folders
Each file folder can contain multiple files
Each file folder has only one file key for file encryption
All the files in the folder are encrypted by the same file key
When do authorization, one user cannot authorize a single file to another user.
When do authorization, one user can only authorize the entire file folder to another user.
	
Use Case:
1. Client-A upload file to an edge.



Retrieve File
Use Case: client retrieve file







Authorize File
Use Case: client A authorized client B to access a edge.

Server Verify User
Use Case: verify client A after token expired (No need to input master password)



Genesis

In the beginning,
1) xxx-xxx-xxxx phone number sign up, V0
2) create O(rganization), V1;
3) create edge <V0, V1>, etype = IS_OWNER
4) create edge <V1, V0>, etype = IS_ADMIN, which means that V0 is able to get V1.key.SK
	or V1.key.SK is encrypted by V0.key.PK,
    For Vx is the owner of V0, once V0 changed owner, Vx is able to remove edge <V1,V0> for V1's access.
   Admin is able to work as the or
5) Invite an admin user.
6) 





Type of Edge, Event,  - eType

Each is edge is only created by bVid and signed by bVid.
Edge producing speed is governed by the system assigned policies to the Vertex.

"TO BE" could infer the relation of "TO HAVE" or "TO DO". Therefore, eType is directly referenced to the English "NOUN", i.e. Naming word. In the beginning, there is word.
VERB is an abstracted action, or a named action.

Each eType should be inferred to unique  set derived attributes of the edge.
eType is the property of an edge to abstract the its unique attributes.
The proliferation of edge (producing) should be governing the law of this document.

0 : Own, Is_Owner, As_Owner
     A is the owner of B. B can only a Vo node.
     Ownership has to be referred by direct edge.
     If A is the owner of B, e = <A, B> and e.etype == OWNER, 
sKeyDecrypt(aKeyDecrypt(A.key.SK,.e.bESAK), B.key.ESK) == B.key.SK
aKeyDecrypt(A.key.SK, e.bESAK) is the master password for access the B.key.ESK.
Then, A is able to sign as B. Anything shared with B is accessible by A.
e.eESAK == null,
e.isPrivate() == true;
e.bESAK = aKeyEncrypt(
    There is only one e.isRoot(), i.e. e.refid == 0.
   Co-owner edge has to be signed by the bVid of the root edge.
   
   

1 : Admin
     e = <A,B>,  e.etype == ADMIN
     A authorizes B to access A.SK
     B has to be a Vp.
     A is able to revoke to authorization be remove the edge.
     B is able to access the SK of A. i.e.
     sKeyDecrypt(aKeyDecrypt(B.key.SK, e.eESAK), A.key.ESK)) == A.key.SK
     aKeyDecrypt(B.key.SK, e.eESAK) is the master password for accessing the A.key.ESK.

    e.eESAK = aKeyEncrypt(B.key.PK, A.key.PW)

    For accessing A.SK, 

2 : Admin 
Trilation is a three-part relationship. If A


For each edge(event) creation, e.name carries the client-side parameters, and e.deat is server's response.
Then, we don't need to extend APIs for support more services(function).
In this way, each request-response is recorded in the chain-of-edge. The etype will be main discrimintae.
request_pk(string siteName)
request_pin(string uid) // such as one-time password or login-token.

etype : requestPin
name : { "uid":"phone_number"}
deat: {"pk_o":"xxxxxxxx"}

etype:requestKey
name : {"uid"":"phone_number";"epin":encrypt(pk_o, "pin"}
deat:{"jwt":""pk", "esk"};
              "


Error Code and Message

    EcosOk EcosError = 0
    //gRPC error and general errors
    NullInputObject      EcosError = 10
    InvalidNameJson      EcosError = 20
    JWTobjectTypeError   EcosError = 110
    JWTNotFound          EcosError = 111
    JWTVCodeError        EcosError = 112
    JWTExpired           EcosError = 113
    JWTPermissionDeny    EcosError = 114
    DBConnectionError    EcosError = 120
    SQLExecError         EcosError = 200
    SQLBeginError        EcosError = 201
    SQLCommitError       EcosError = 202
    SQLResultError       EcosError = 205
    SQLStatementGenError EcosError = 210
    //Select Error 220
    //Insert Error 230
    //Update Error 240
    SQLUpdateWithoutId EcosError = 240
    //Delete Error 250
    TwilioConnectionError        EcosError = 300
    TwilioResponseError          EcosError = 310
    NotImplementedMultipleDelete EcosError = 400
    //Vertex related error
    InvalidUserId         EcosError = 1000
    CannotFindUserId      EcosError = 1010
    CannotFindUid         EcosError = 1020
    UidHasBeenUsed        EcosError = 1030
    UserIDNotMatch        EcosError = 1040
    UidIsEmpty            EcosError = 1050
    CreateVertexWrongId   EcosError = 1060
    InvalidPublicKeyLen   EcosError = 1070
    InvalidESeceretKeyLen EcosError = 1071
    DeleteIdNotFound      EcosError = 1072
    CreateDocInvalidEid   EcosError = 2000
    CannotFindHostEdge    EcosError = 2010
    UpdateDocIdNotFound   EcosError = 2070
    DeleteHostIdNotFound  EcosError = 2072
    //Edge related error
    CreateDocWrongId     EcosError = 2060
    InvalidEType         EcosError = 3000
    DeleteObjHasChild    EcosError = 3001
    NoVerificationCode   EcosError = 3010
    CreateEdgeWrongId    EcosError = 3060
    UpdateEdgeIdNotFound EcosError = 3070

  In java,  INTERNAL: JWT Verification Code Error, could be due to not setting the verification code in vertex's tag field or not setting the jwt in request


   CannotFindHostEdge:           "Create Doc Cannot Find Host Edge",
    CannotFindUid:                "Cannot Find by UID",
    CannotFindUserId:             "Cannot Find user_id",
    CreateDocInvalidEid:          "Create Doc with Invalid Edge Id",
    CreateDocWrongId:             "Create Doc With a Wrong Id",
    CreateEdgeWrongId:            "Create Edge With a Wrong Id",
    CreateVertexWrongId:          "Create Doc With a Wrong Id",
    DBConnectionError:            "Cannot connect to DB",
    DeleteHostIdNotFound:         "Cannot find the host ID of a Delete item",
    DeleteIdNotFound:             "Delete ID not found",
    DeleteObjHasChild:            "Delete Item which has child(ren)",
    EcosOk:                       "ECOS OK",
    InvalidESeceretKeyLen:        "Invalid Encrypted Secret Key Length, expected 72",
    InvalidEType:                 "Invalid etype, not implemented yet",
    InvalidNameJson:              "Invalid name Json",
    InvalidPublicKeyLen:          "Invalid Public Key Length, expected 32",
    InvalidUserId:                "Invalid user_id",
    JWTExpired:                   "JWT Expired",
    JWTNotFound:                  "JWT is not found, may have been expired",
    JWTPermissionDeny:            "JWT Permission Deny",
    JWTVCodeError:                "JWT Verification Code Error",
    JWTobjectTypeError:           "Mismatched JWT object type",   // you can get this error when creating notebook but did not set up bvid
    NoVerificationCode:           "No Verication Code in name of edge",
    NotImplementedMultipleDelete: "Multiple id deletion has not implemented",
    NullInputObject:              "Nil Input Object",
    SQLBeginError:                "SQL Begin Transaction Error",
    SQLCommitError:               "SQL Commit Transaction Error",
    SQLExecError:                 "SQL Execution Error",
    SQLResultError:               "SQL Result Processing Error",
    SQLStatementGenError:         "SQL Error When Generate Statement",
    SQLUpdateWithoutId:           "Update Without ID",
    TwilioConnectionError:        "Twilio Http Connection Error",
    TwilioResponseError:          "Twilio Http Response Error",
    UidHasBeenUsed:               "Uid Has Been Used",
    UidIsEmpty:                   "Uid Is Empty",
    UpdateDocIdNotFound:          "Update Doc, ID is not found",
    UpdateEdgeIdNotFound:         "Update Edge, ID is not found",
    UserIDNotMatch:               "UserId not match",


Global Config

https://config.aitmed.com/{app_name}.json

app_name ::= prynote | aitmed

fallback logic, if app_name.json is not presented, using aitmed.json as fallback

aitmed.json:
{
 "apiHost":"https://ecosapi.aitmed.com"
}
apiPort: default https://ecosapi.aitmed.com
if access is not available, try "http://ecosapi.aitmed.com"
if it start with "http://" use port=8
if is

app_name is the subdomain of aitmed.com, exp prynote.aitmed.com, the prynote is an app_name
app_name is the domain of non-aitmed.com. exp, acurgentcare.com, the "acurgentcare" is the app_name.

1) https://config.aitmed.com/{app_name}.json
if not presented
2) http://config.aitmed.com/{app_name}.json
if not presented
3)https://config.aitmed.com/aitmed.json
if not presented
4)http://config.aitmed.com/aitmed.json




Push Notification

General Notification will be sent for a created docObj.

General Ecos File System and Access Right

If docObj.eid points to an edgeObj, this docObj is a root docObj. docObj is equivalent to iNode of a UNIX/LINUX file system.
If A docObj.eid points B docObj, B docObj is the folder for the A docObj. 
Accessible user(Vertex) is defined by a group for edge which are have the same "refid",
among them, one edgeObj.ID == edgeObj.Refid is the root edge object. All vertex(User) on the group share the same SAK(Session Access Key) for the files, which Reid(Esig) is the rootEdgeObj.ID.
When docObj is created, frontend(Noodl) only specifies the "Eid" of the docObj, it is upto the ecos_server to find and assign the Reid to the object.
When frontend needs to find the SAK to decrypt a docObj, frontend uses docObj.Reid==edgeObj.Refid to find the edgeObj which holds the xESAK (x in {b,e} and E stands for encrypted) for the user to decrypt the xESAK to SAK.



How to find authEdge, which contains xESAk;
if doc.eid is EdgeID {
   authEdge = getEdgeByID(doc.eid)
} else {
   authEdge = getEdgeByID(doc.reid)
}


Therefore, for any created docObj, ecos-server will send notification for all vertice on the group of the edgeObj. All vertice are including the person who creates the docObj. Because, the notification will be sent to all device of the person. This will enable the cross-device synchronization for the same person.










Austin shared algorithm to determine if object is edge, doc, vertex from Object ID


We are using google firebase https://firebase.google.com/docs/cloud-messaging/android/client

Appid is SHA256(appName), take the first half = 128 bit, become byte array
AppName is defined in the app's base Data Model


1090 edge's purpose is to give the firebase access token to backend

	bvid is from jwt
	evid is the appid
	Everytime our app has a local credential, we send an edge 1090 to give the token to the backend.
           refid is the SHA128(first half of SHA256) of the FCM Token, which is stored in the name field.

  1091 (subtype = 0) for 1 to 1 chat

 	This edge set up the chat relationship between inviter and invitee

	bvid is from jwt
	evid is the invitee (from name's phone number)
	refid is appid
	All messages between the 2 users will be a doc with eid to this 1091 edge.

  1091 (subtype = 1) for group chat


    Every invite to another user into this group will be another 1091 edge
    The first inviter send 1091 edge to an invitee with (subtype = 1) is the group chat root edge

    For group chat root edge:
    	bvid is from jwt
		evid is the invitee (from name's phone number)
		refid is null

    	All messages in this group chat will be documents attached to this group chat root edge

    	To retrieve documents in group chat, use this edge's id as eid to get docs

    For later 1091 edges in group chat,
      	bvid is from jwt
		evid is the invitee (from name's phone number)
		refid is the group chat root edge

		To retrieve documents in group chat, use refid as eid to get docs

To handle multiple devices:

Each user will have his own access token, this access will change on app-reinstall or clean app data.
New message will send to all devices

All device after having local credential will create a 1090 edge.


If one device previously created 1090 edge already, and then reinstall again, another 1090 edge will be created.

Backend will check mtime and only use the recent edges within 24 hour, so the old access token will not be use eventually.

When user logout, user should not receive notification anymore.



It is possible for app to choose not to receive notification anymore by deleting the firebase instance or delete the token

https://firebase.google.com/docs/reference/android/com/google/firebase/iid/FirebaseInstanceId#deleteInstanceId()

But not recommended
https://stackoverflow.com/questions/40589593/restricting-gcm-fcm-notifications-through-unregister?noredirect=1&lq=1


To maintain the participant list in a group chat
Every time a new person get added, front end will attach a document with type = 5 to the current edge with the invitee’s name
Front end still need to add the document with type = 5 for each invited edge (sync)
To get invited group list
Problem: Currently to retrieve all the chat group this user is in, xfname is both evid|bvid, so all the 1091 invite edge will also get retrieved

Solution:  use xfname = “refid = id AND bvid|evid”  




To get documents in a list
Documents are now stored inside the invited edge because backend has special join logic to retrieve as a document group.

For return all documents of a group chat
api:rd
ObjType:4
ids: set to the rootEdgeID
Xfname:E.refid
Type:1091
Obfname:D.mtime










Personal file folders(edge.Type=10000) only hold files which are only accessed by the User. It contents are encrypted by the key which is encrypted by the user's pk(public key). It only could be access the users' secretKey.

Each user has only one special Person file fold, the Root Note Book. ItFront End Level2 SDK creates encryption key for Notebook.name contains {"isEncrypted":"1"} 

Edge.isEncrypted !== Edge.Subtype & 1 == 0

if Edge.isEncrypted && Edge.bESAK == nil {
     Edge.bESAK = AESKey encrypted by users.Vertex.pk
     call api 'cd' with its Edge.ID to update the Edge in ECOS
}
s subtype is defined as "1". This is an anonymous file folder holds unclassified(uncategorized) documents.
Then, in this folder, documents are grouped by it type.
As right now, ECOS has defined following document types

Every user only has one Root Notebook, which is created when a user first time signIn.
Its id(eid) is stored in userVertex.deat as {"rnb64ID":"xxxxxxxxxxxx=="}
rnb64ID stands for Root Notebook Base64 formatted ID.

rootNotebook Document type:
	 type: “1” # profile       (Edit Profile, DisplayProfile)
             type: “2” # contact     (AddContact, ContactsList, ContactInformation, EditContact)
             type: “3” # favorite contact  (FavoriteList)
             type: “4” # other Documents (AddDocuments, DocumentDetail, PersonalDocuments, UploadDocuments)

Let medical application, uncategorized document.type starts from 10000.
doc.Type=10000 is patientChart


 
SignIn/SignUp Logic


Document Types

Reserve lower 8 bits in document type for bitwise attributes
bit0 = 1 encrypted
bit1 = 1 favorite

starts from bit8 above, use it scala value to indicate different types.

DocType:
 1: PubProfile:    "256" #              0x100 			
 1: Profile:          "257"                  0x101  private, 		
 2: Contact:       "513"                   0x201, private
 2: ContactFav:  "515"                  0x203 private | favorite
 3: short Message  "769"       0x301  // binary is 1100000001, shift 8 bit left becomes 11, which is 3
  GetAllContact: "type in (513,515)"
  GetFavContact: "type=515"
4: UploadFile:   "1025"                  0x401  private updated document
5: InviteUserMessage: "1281"      0x501  private invitation
6: MeetingNote: "1537" # 6<<8+1 6
7: Mail Message    "1793"  # 7<<8+1   0x701 message with subject line. hexical 

b1010   binary
0777     octal 8
1234     decimal
0xFFFF hexical

Type=40000 meeting room scheduling logic

Meeting Room scheduling live-cycle

1. Start from set availability by create edge E3 = {type=40000, subtype=3}.
2. Invite a user by create edge I1053 = {type=1053, refid = E3.id}
3. Patient get doctor's availability by
   a) From doctor in contact
       rv = {bvid=doctor.id, type=40000, subtype=1 and stime>the_day_start_time
               etime< the_day_end_time}
   b) From doctor calendar invite by
      rv = {refid=E3.id, type=40000 subtype=1 and stime>the_day_start_time and
              etime<the_day_end_time}
4 Request an appointment by create edge E2 = {type=40000 and subtype=2 and
             refid = the_selected_time_slot_edge.id
     And attach documents to the edge
5 Doctor accept invite by create edge E5={type=40000,subtype=5,refid=E2.id}
   after doctor accept or confirmed an appointment
   E2.subtype = 6(confirmed appointment), this edge become the meeting root edge
   for creating


 meeting_root(type=40000, subtype=0 and refid=E2} since all document is saved
  here. And make E2.refid retired(the select time_slot).
6 Doctor reject an appointment request by create edeE E7 = {type=40000 and subtype=7
  refid = E2.id
  E2 will be retired. by this edge creation.
7. Doctor block a specific time_slot by create edge E4_b1={type=40000 and subtype=4,
refid=the _selected_time_slot
    If refid=nil, using the name field to carry range setting option to block broad period.
8. Reschedule an confirmed(subtype=6) appointment by create ede(subtype=8)
9. If one request is accepted, other requests on the same time slot will be marked as
    "Conflicted(subtype=9)

For subtype=3 and 7, it only applies to future time slots. only subtype=1(available) time_slot could be retired as this action. If there are subtype=2(available) or subtype=6(scheduled)
exist, error message will return to inform the user about schedule conflict.
If tage=-1(forcefully), all conflict schedules will be canceled and the users will be notified for the cancellation.

Only subtype=2(request) could be change to subtype=6(confirmed) and liveMeetingRoom is          created by referencing to the subtype=6 as appointment-meeting-root, which holds all documents.






Notification Processing Queue
There are three threads handling the notification delivering process.
1) NQ thread will start when the ecos server is started. It will stop when the ecos-server is shutdown. Unexpected stop of this Queue is considered as BUG(unexpected stopped)
This thread is listening and blocked on the notificationChannel. If any dObj is received, the dObj is dispatched for immediate delivery.

2) TQ is a timed Queue. It buffered up-to the capacity of its buffer, (currently 100).
Its future notification dObj has a

scale := int64(etypes.MicroSecond2Second) // 1000000
sendTime=(dm.Ctime+int64(dm.Tage)-TQLeadTime)*scale + dm.Mtime%scale
TQLeadTime is not set to 60 seconds.

The dm.Mtime%scale is used to distinguish the notifications are felled within one second.

It picked up the earliest(sendTime) notification dObj from the buffer and set the sleep timer to wake to put the dObj into the notificationChannel for delivering.

3) When a notification request is received from API, (normally from other thread), if the notification is not scheduled (future) notification, it will be put into the notification channel immediately. If it is future notification, it will be put into the processChannel of the TQ thread.
Once TQ thread received the future dObj, if the TQ.buffer is full and the dObj's sendTime is later than the maxSendTime of the objects in the buffer. It will throw away for future pickup.
If the TQ.buffer is full and the dObj's sendTime is earlier than the maxSendTime of the objects in the buffer. The latest item in the buffer is replaced with the new dObj. If the TQ.buffer has an empty spot, the new dObj will be put there. Then, the min and max sendTime of the objects in the buffer is updated. 

4) The third thread monitors and updates the TQ.buffer. This is a timer thread. It sleeps until the earliest scheduled notification to send. The monitoring loop can be interrupted by timeoutChannel if a new notification object is earlier than the earliest notification object. Then the timer is set according to the new earliest time.

5) If the TQ.buffer is empty, the third timer thread will be stopped. And it will be restarted when a item is add to the buffer for future delivery.

6) If dm.Tage < 0, this dm has been sent.

7) Fetch notification dm from DB when
    a) NQ is started
    b) TQ.buffer item count is less than 1/3 of TQ capacity 
8) If nothing returned by fetch() from DB, no more fetch() in next update()
    And fetch() is resumed if any future item is dropped out because its sendTime is greater than
    TQ.maxSendTime.




Database cache and update Queue

v, e, d objects are cached in memory for quick access. If an object is updated, its updated result will be returned after the updating is successful. In the meantime, this updated object is also saved into the cache for future queries. For rx query, if the xfname is empty, by default it is 'id",
ecos will return it from its cache. If cache-miss, a SQL is used to obtain it from DB.

All DB updates/inserts are executed in batch SQL transactions by a single thread. In this way, Any API required updates/inserts can be processed before the SQLs were executed. This will greatly improve the API response time.


Database Table Spawn

No Data removed from Table via "Dx" API.
Dx API only update the "Type" from positive one to "Negative".
Metabolism algorithm will move the "Negative" typed object to Table2, as "trash can" table.

Table is the live table which is constantly updated by Ecos APIs.

Table0 is a snapshot of Table in a given moment.

Table01 is a data removed from Table via metabolism. It is considered as the extension of Table to store non-mission critical data to increase the efficiency of Table. Most data moved from Table to Table01 is time-based.

Table and Table01 should contain only "positive" typed objects.
There is no duplicated ID in the combined "Table" and Table01.

Table02 is used to backup data before it is updated. Everytime a data is backed up from Table to Table02. The previous version of the data is overwritten.

Table and Table1 are the same. Table is alias of Table1










