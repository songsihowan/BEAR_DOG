
# 🕹️ BEAR AND DOG

곰, 강아지 캐릭터가 보물을 되찾기 위해 여러지형을 탈출하는 게임

## 📌 소개 (Introduction)

이 프로젝트는 Unity으로 개발된 3D 2인 멀티 게임입니다.  
플레이어는 보물을 목표로 2개의 스테이지를 클리어해야 합니다.

## 🚀 주요 기능 (Features)


<details>
  <summary>🔍 방 생성 및 삭제 시스템 (열기/닫기)</summary>

```csharp
// 접속순서는 번호로 적음
    void Awake()
    {
        if(!PhotonNetwork.IsConnected) 
        // Photon 서버 접속 확인 (인게임에서 빠져나온 경우 재접속 필요)
        {
            //1번, 포톤 클라우드에 접속 시도
            PhotonNetwork.ConnectUsingSettings();
            // Photon 서버 접속 시도 (지역 서버 접속 및 AppId 인증 포함)
        }
        //사용자 이름 설정
        UserId.text = GetUserId();

        //룸 이름을 무작위로 설정
        roomName.text = "Room_" + Random.Range(0, 999).ToString("000");
    }

    // Start is called before the first frame update
    void Start()
    {
        PhotonNetwork.ConnectToRegion("kr"); // 한국 서버(Region) 직접 연결

        if (UserIdBtn != null)
            UserIdBtn.onClick.AddListener(OnClickNameRandomRoom);

        if (CreateRoomBtn != null)
            CreateRoomBtn.onClick.AddListener(OnClickCreateRoom);
    }

    // Update is called once per frame
    void Update()
    {
        if (0.0f < ShowMsTimer)
        {
            ShowMsTimer -= Time.deltaTime;
            if (ShowMsTimer <= 0.0f)
            {
                MessageOnOff("", false); //메시지 숨김
            }
        }//if(0.0f < ShowMsTimer)     
    }

    //2번, ConnectUsingSetting() 함수 호출에 대한 서버 접속이 성공하면 호출되는 콜백 함수
    public override void OnConnectedToMaster()
    {
        Debug.Log("포톤 서버 접속 완료");
        // Master 서버 접속 완료 (PhotonNetwork.ConnectToMaster 상태) 

        //3번, 규모가 작은 게임에서는 서버 로비가 보통 하나이고...
        //대형 게임인 경우 상급자로비, 중급자로비, 초보자로비처럼
        // 여러 로비(상급자, 중급자, 초보자 등) 존재 가능
        PhotonNetwork.JoinLobby();  //포톤에서 제공해 주는 가상의 로비에 접속 시도
    }

    //4번, PhotonNetwork.JoinLobby() 성공시 호출되는 로비 접속 콜백함수
    public override void OnJoinedLobby()
    {
        if (isWaitingToCreateRoom)
        {
            CreateRoom();
            isWaitingToCreateRoom = false;
        }
    }
    public void CreateRoom()
    {
        string _roomName = roomName.text;
        //룸 이름이 없거나 null 일 경우 룸 이름 저장
        if (string.IsNullOrEmpty(roomName.text))
        {
            _roomName = "Room_" + Random.Range(0, 999);
        }

        PhotonNetwork.LocalPlayer.NickName = UserId.text;
        //생성할 룸의 조건 설정
        RoomOptions roomOptions = new RoomOptions();
        roomOptions.IsOpen = true;      //입장 가능 여부
        roomOptions.IsVisible = true;   //로비에서 룸의 노출 여부
        roomOptions.MaxPlayers = 2;     //룸에 입장할 수 있는 최대 접속자 수

        //지정한 조건에 맞는 룸 생성 함수
        PhotonNetwork.CreateRoom(_roomName, roomOptions, TypedLobby.Default);
    }

    public override void OnJoinedRoom()
    {  
        //서버 역할인 경우 5번 : 방입장, 클라이언트 역할인 경우 4번 : 방입장

        Debug.Log("방 참가 완료");

        //룸 씬으로 이동하는 코루틴 실행
        StartCoroutine(this.LoadBattleField());
    }

    //룸 씬으로 이동하는 코루틴 함수
    IEnumerator LoadBattleField()
    {
        //씬을 이동하는 동안 포톤 클라이드 서버로부터 네트워크 메시지 수신 중단
        PhotonNetwork.IsMessageQueueRunning = false;
        //백그라운드로 씬 로딩
        AsyncOperation ao = SceneManager.LoadSceneAsync("SelectcharacterScene");

        yield return ao;
    }

    //PhotonNetwork.JoinRandomRoom(); 함수가 실패한 경우 호출되는 오버라이드 함수
    public override void OnJoinRandomFailed(short returnCode, string message)
    {
        Debug.Log("랜덤 방 참가 실패 (참가할 방이 존재하지 않습니다.)");

        //방을 생성하면서 들어감
        //생성할 룸의 조건부터 설정
        RoomOptions roomOptions = new RoomOptions();  //using Photon.RealTime;
        roomOptions.IsVisible = true;   //로비에서 룸의 노출 여부
        roomOptions.BroadcastPropsChangeToAll = true;
        roomOptions.MaxPlayers = 2;     //룸에 입장할 수 있는 최대 접속자 수

        //지정한 조건에 맞는 룸생성 함수
        PhotonNetwork.CreateRoom("MyRoom", roomOptions);
    }

    ////방 생성 실패시 호출되는 함수(대부분 룸의 이름이 이미 존재하는 경우)
    //public override void OnCreateRoomFailed(short returnCode, string message)
    //{
    //    Debug.Log(returnCode + " : " + message);
    //}

    //로컬에 저장된 플레이어 이름을 반환하거나 생성하는 함수
    string GetUserId()
    {
        string userId = PlayerPrefs.GetString("USER_ID");

        if(string.IsNullOrEmpty(userId))
        {
            userId = "USER_" + Random.Range(0, 999).ToString("000");
        }

        return userId;
    }

    //Join Random Room 버튼 클릭 시 호출되는 함수
    public void OnClickNameRandomRoom()
    {
        //로컬 플레이어의 이름을 설정
        PhotonNetwork.LocalPlayer.NickName = UserId.text;
        MessageOnOff(UserId.text+"로 닉네임을 설정했습니다.");


        //플레이어 이름을 저장
        PlayerPrefs.SetString("USER_ID", UserId.text);

    }

    //Make Room 버튼 클릭 시 호출될 함수
    public void OnClickCreateRoom()
    {
        if (!PhotonNetwork.InLobby)
        {
            Debug.Log("로비에 아직 안 들어감. 대기 중...");
            isWaitingToCreateRoom = true;
            return;
        }
        CreateRoom();

    }

    //PhotonNetwork.CreateRoom() 함수 호출이 실패 하면 호출되는 함수
    //(같은 이름의 방이 이미 있을 때 실패함)
    public override void OnCreateRoomFailed(short returnCode, string message)
    {
        Debug.Log("방 만들기 실패");
        MessageOnOff("방만들기에 실패했습니다 다시 입력하세요");
        //주로 같은 이름의 방이 존재할 때 룸 생성 에러가 발생된다.
        Debug.Log(returnCode.ToString());  //오류코드(ErrorCode 클래스)
        Debug.Log(message);  //오류 메시지
    }

    public void OnClickRoomItem(string roomName)
    {
        PhotonNetwork.LocalPlayer.NickName = UserId.text;

        //인자로 전달된 이름에 해당하는 룸으로 입장
        PhotonNetwork.JoinRoom(roomName);
    }

    public override void OnRoomListUpdate(List<RoomInfo> roomList)
    {
        /*
        새 방 생성 or 기존 방 정보 갱신:
        - 방이 새로 만들어지면 RoomItem 프리팹 인스턴스 생성
        - 기존 방이라면 RoomItem 정보 갱신
        */

        m_RoomItemList = scrollContents.transform.GetComponentsInChildren<RoomItem>(true);

        int roomCount = roomList.Count;
        int ArrIdx = 0;
        for(int i = 0; i < roomCount; i++)
        {
            ArrIdx = MyFindIndex(m_RoomItemList, roomList[i]);

            if (roomList[i].RemovedFromList == false)
            { 
                if(ArrIdx < 0)
                { //방을 새로 생성하는 경우
                    GameObject room = Instantiate(roomItem) as GameObject;  
                    //생성한 RoomItem에 프리팹의 Parent를 지정
                    room.transform.SetParent(scrollContents.transform, false);
                    //생성한 RoomItem에 표시하기 위한 텍스트 정보 전달
                    RoomItem roomData = room.GetComponent<RoomItem>();
                    roomData.roomName = roomList[i].Name;
                    roomData.connectPlayer = roomList[i].PlayerCount;
                    roomData.maxPlayer = roomList[i].MaxPlayers;

                    //텍스트 정보를 표시
                    roomData.DispRoomData(roomList[i].IsOpen);
                }
                else  //해당 방이 리스트 뷰에 존재하면 방정보만 갱신해 줘야 한다는 의미
                {  
                    //기존 방 정보만 갱신
                    m_RoomItemList[ArrIdx].roomName = roomList[i].Name;
                    m_RoomItemList[ArrIdx].connectPlayer = roomList[i].PlayerCount;
                    m_RoomItemList[ArrIdx].maxPlayer = roomList[i].MaxPlayers;

                    //텍스트 정보를 표시
                    m_RoomItemList[ArrIdx].DispRoomData(roomList[i].IsOpen);
                }
            }
            else 
            { //방이 파괴되어서 리스트뷰에서 RoomItem을 제거
                if(0 <= ArrIdx)
                {
                    MyDestroy(m_RoomItemList, roomList[i]);
                    //이 방 정보를 갖고 있는 리스트뷰 목록을 모두 제거
                }
            }
        }
    }

    int MyFindIndex(RoomItem[] RmItemList, RoomInfo roomInfo)
    {
        if(RmItemList == null)
            return -1;

        if(RmItemList.Length <= 0)
            return -1;

        for(int i = 0; i < RmItemList.Length; i++)
        {
            if(RmItemList[i].roomName == roomInfo.Name)
            {
                return i;
            }
        }

        return -1;
    }

    void MyDestroy(RoomItem[] RmItemList, RoomInfo roomInfo)
    {
        if (RmItemList == null)
            return;

        if(RmItemList.Length <= 0)
            return;

        for(int i = 0; i < RmItemList.Length; i++)
        {
            if (RmItemList[i].roomName == roomInfo.Name)
            {
                Destroy(RmItemList[i].gameObject);
            }
        }
    }
```
</details>

<details>
  <summary>🔍 캐릭터 선택 동기화 코드 (열기/닫기)</summary>

```csharp
public enum SelectState //게임상태 이너명 설정
{
    Select,
    GameReady,
}

public enum Team // 캐릭터 설정
{
    Select,
    Dog,
    Bear,
}
public class SelectMgr : MonoBehaviourPunCallbacks
{
    public Button Back_Btn;

    public RectTransform dogSpawnPoint;    // 강아지 팀 Ui이동할 위치
    public RectTransform bearSpawnPoint;   // 곰 팀 Ui이동할위치
    public RectTransform selectSpawnPoint; // 선택 팀 Ui이동할위치
    public RectTransform canvasTransform;

    public GameObject playerUI; // 플레이어의 UI 오브젝트(프리팹)

    public Team selectedTeam = Team.Select; // 현재 플레이어가 선택한 팀 기본적으로는 선택할 수 있게

    ExitGames.Client.Photon.Hashtable m_SelTeamProps = new ExitGames.Client.Photon.Hashtable();

    ExitGames.Client.Photon.Hashtable m_PlayerReady = new ExitGames.Client.Photon.Hashtable();

    private float moveSpeed = 5f; // UI 이동 속도

    // 리스트로 관리 (UI 오브젝트 및 목표 위치)
    private List<GameObject> playerUINodes = new List<GameObject>();
    private List<Vector2> targetPositions = new List<Vector2>();

    bool ReadOnOff = false;

    public static SelectMgr inst = null;


    [HideInInspector] public float m_GoWaitGame = 4.0f;     //게임 시작 후 카운트 Text UI
    [HideInInspector] public float m_MgWaitGame = 2.0f;     //게임 시작 후 카운트 Text UI

    public Text m_WaitTmText = null;
    public Text RoomNameText = null;
    public Text MassageText = null; 

    bool allReady = false;

    private bool isPhotonReady = false;

    void Awake()
    {
        inst = this;
        PhotonNetwork.IsMessageQueueRunning = true;
        RoomNameText.text =PhotonNetwork.CurrentRoom.Name;
    }

    void Start()
    {
        StartCoroutine(WaitAndInitProps());

        if (Back_Btn != null)
        {
            Back_Btn.onClick.AddListener(OnclickBackBtn);
        }

        UpdatePlayerUI();
    }
    IEnumerator WaitAndInitProps()
    {
        while (!PhotonNetwork.InRoom || PhotonNetwork.NetworkClientState != ClientState.Joined)
            yield return null;

        InitReadyProps();
        isPhotonReady = true;
    }
    void Update()
    {
        // UI 리스트의 인덱스 순서대로 부드럽게 이동 처리
        for (int i = 0; i < playerUINodes.Count; i++)
        {
            RectTransform rectTransform = playerUINodes[i].GetComponent<RectTransform>();
            Vector2 currentPosition = rectTransform.anchoredPosition;
            Vector2 targetPosition = targetPositions[i];

            rectTransform.anchoredPosition = Vector2.Lerp(currentPosition, targetPosition, Time.deltaTime * moveSpeed);
        }

        if (PhotonNetwork.IsConnected && PhotonNetwork.InRoom || allReady==false)
        {

            // 키 입력을 통해 팀 선택 처리
            if (Input.GetKeyDown(KeyCode.LeftArrow) && selectedTeam == Team.Select)
            {
                SelectTeam(Team.Dog);
            }
            if (Input.GetKeyDown(KeyCode.LeftArrow) && selectedTeam == Team.Bear && ReadOnOff == false)
            {
                SelectTeam(Team.Select);
            }
            if (Input.GetKeyDown(KeyCode.RightArrow) && selectedTeam == Team.Select)
            {
                SelectTeam(Team.Bear);
            }
            if (Input.GetKeyDown(KeyCode.RightArrow) && selectedTeam == Team.Dog && ReadOnOff == false)
            {
                SelectTeam(Team.Select);
            }
            if (Input.GetKeyDown(KeyCode.E) && (selectedTeam == Team.Dog || selectedTeam == Team.Bear))
            {
                ReadOnOff = !ReadOnOff;
                if (ReadOnOff)
                {
                    SendReady(1);
                }
                else
                {
                    SendReady(0);
                }
            }
        }
        if (0.0f < m_MgWaitGame) //타이머 카운트 처리
        {
            m_MgWaitGame -= Time.deltaTime;
            if (MassageText != null)
            {
                MassageText.gameObject.SetActive(true);
            }
            if (m_MgWaitGame <= 0.0000001f)
            {
                MassageText.gameObject.SetActive(false);

            }

        }
        if (!PhotonNetwork.InRoom) return; // 방에 없으면 실행 안 함
        AllReadtObserver();

    }


    private void SelectTeam(Team team)
{
    if (!isPhotonReady)
    {
        Debug.LogWarning("Photon이 아직 준비되지 않아서 팀 선택 불가");
        return;
    }

    if (selectedTeam == team)
    {
        MassageText.text = "같은팀을 선택했어요!";
        return;
    }

    selectedTeam = team;

    Hashtable playerProperties = new Hashtable { { "MyTeam", team.ToString() } };
    PhotonNetwork.LocalPlayer.SetCustomProperties(playerProperties);
    Debug.Log($"팀 {team}을(를) 선택했습니다.");
}
    // UI 업데이트 및 재생성을 담당하는 함수
    void UpdatePlayerUI()
    {
        Player[] players = PhotonNetwork.PlayerList;

        // 플레이어 수가 변경되었을 때만 UI 오브젝트를 재생성합니다.
        if (playerUINodes.Count != players.Length)
        {
            // 기존 UI 오브젝트들을 제거 (플레이어 수가 달라졌을 때만)
            foreach (GameObject ui in playerUINodes)
            {
                Destroy(ui);
            }
            playerUINodes.Clear();
            // 리스트를 새롭게 생성
            foreach (Player p in players)
            {
                
                GameObject userNode = Instantiate(playerUI, canvasTransform);
                userNode.name = $"UserNode_{p.NickName}";
                playerUINodes.Add(userNode);
            }
        }

        // 새 목표 위치 리스트를 계산하고 업데이트 (UI 오브젝트는 그대로 두어 Lerp 효과를 줍니다)
        targetPositions.Clear();
        for (int i = 0; i < players.Length; i++)
        {
            Player p = players[i];
            GameObject ui = playerUINodes[i];
            PlayerInfo playerInfo = ui.GetComponent<PlayerInfo>();

            string team = p.CustomProperties.ContainsKey("MyTeam") ? p.CustomProperties["MyTeam"].ToString() : "None";
            RectTransform targetParent = null;
            bool isMine = (playerInfo.m_UniqId == PhotonNetwork.LocalPlayer.ActorNumber);
            bool isReady = ReceiveReady(p); // 🔹 레디 상태 가져오기

            if (team == "Dog")
            {
                targetParent = dogSpawnPoint;
                playerInfo.DispPlayerData(p.NickName, isMine, true, false, isReady);
                
            }
            else if (team == "Bear")
            {
                targetParent = bearSpawnPoint;
                playerInfo.DispPlayerData(p.NickName, isMine, false, true, isReady);
            }
            else
            {
                targetParent = selectSpawnPoint;
                playerInfo.DispPlayerData(p.NickName, isMine, false, false, isReady);
            }

            if (targetParent != null)
            {
                Vector2 basePosition = targetParent.anchoredPosition;
                // 예시: 두 번째 플레이어이면 Y 오프셋 추가
                if (p.ActorNumber == 2)
                {
                    basePosition.y += 200f;
                }
                targetPositions.Add(basePosition);

               
            }
        }

    }

    public override void OnPlayerPropertiesUpdate(Player targetPlayer, Hashtable changedProps)
    {
        if (changedProps.ContainsKey("MyTeam"))
        {
            string updatedTeam = changedProps["MyTeam"].ToString();
            Debug.Log($"{targetPlayer.NickName}님이 {updatedTeam} 팀에 합류했습니다.");
            // 팀 선택이 변경될 때마다 UI를 갱신하면
            // 기존 UI 오브젝트들을 Destroy하고 새로 생성됩니다.
            UpdatePlayerUI();
        }
        if(changedProps.ContainsKey("IamReady"))
        {

            // 팀 선택이 변경될 때마다 UI를 갱신하면
            // 기존 UI 오브젝트들을 Destroy하고 새로 생성됩니다.
            UpdatePlayerUI();

        }
    }

    void OnclickBackBtn()
    {
        if (PhotonNetwork.PlayerList != null && PhotonNetwork.PlayerList.Length <= 1)
        {
            if (PhotonNetwork.CurrentRoom != null)
                PhotonNetwork.CurrentRoom.CustomProperties.Clear();
        }
        if (PhotonNetwork.LocalPlayer != null)
            PhotonNetwork.LocalPlayer.CustomProperties.Clear();
        PhotonNetwork.LeaveRoom();
    }

    public override void OnLeftRoom()
    {
        Time.timeScale = 1.0f;
        SceneManager.LoadScene("LobbyScene");
    }

    // 로컬 플레이어의 인덱스를 PhotonNetwork.PlayerList 순서대로 찾는 함수
    int GetLocalPlayerIndex()
    {
        Player[] players = PhotonNetwork.PlayerList;
        for (int i = 0; i < players.Length; i++)
        {
            if (players[i] == PhotonNetwork.LocalPlayer)
                return i;
        }
        return -1;
    }
    void InitReadyProps()
    {   //속도를 위해 버퍼를 미리 만들어 놓는다는 의미
        m_PlayerReady.Clear();
        m_PlayerReady.Add("IamReady", 0);   //기본적으로 아직 준비 전 상태로 시작한다.
        PhotonNetwork.LocalPlayer.SetCustomProperties(m_PlayerReady); //캐릭터별 동기화
    }

    void SendReady(int a_Ready = 1)
    {
        if (m_PlayerReady == null)
        {
            m_PlayerReady = new ExitGames.Client.Photon.Hashtable();
            m_PlayerReady.Clear();
        }

        m_PlayerReady["IamReady"] = a_Ready;

        PhotonNetwork.LocalPlayer.SetCustomProperties(m_PlayerReady); //캐릭터별 동기화
    }

    bool ReceiveReady(Player a_Player)
    {
        if (a_Player == null)
            return false;

        if (a_Player.CustomProperties.ContainsKey("IamReady") == false)
            return false;

        if ((int)a_Player.CustomProperties["IamReady"] == 1)
            return true;

        return false;
    }

    void AllReadtObserver()
    {
        bool a_AllReady = true;
        Team? team1 = null; // nullable enum enum을 널로 배치하고 싶을때 사용하면 된다 신기방기
        Team? team2 = null;
        foreach (Player a_RefPlayer in PhotonNetwork.PlayerList)
        {
            if (PhotonNetwork.CurrentRoom.PlayerCount < 2)
            {
                return;
            }
            if (ReceiveReady(a_RefPlayer) == false)
            {
                a_AllReady = false;
                break;
            }
            if (a_RefPlayer.CustomProperties.ContainsKey("MyTeam"))
            {
                string teamStr = a_RefPlayer.CustomProperties["MyTeam"].ToString(); // 팀을 1을넣고 다음에 팀2를 넘어차피 같은팀인지 아닌지 체크하는거라 이렇게만듬
                if (team1 == null)
                {
                    team1 = (Team)System.Enum.Parse(typeof(Team), teamStr);
                }
                else if (team2 == null)
                {
                    team2 = (Team)System.Enum.Parse(typeof(Team), teamStr);
                }
            }
        }//foreach(Player a_RefPlayer in PhotonNetwork.PlayerList)
        if (!a_AllReady) return;

        if (team1 != null && team2 != null && team1 == team2)
        {
            m_MgWaitGame = 3.0f;
            MassageText.text = "같은 팀으로는 시작할 수 없습니다!";
            return;
        }
        if (a_AllReady == true)  //모두가 준비 버튼을 누르고 기다리고 있다는 뜻
        {
            allReady = true;    
            //누가 발생시켰든 동기화 시키려고 하면...
            if (PhotonNetwork.CurrentRoom.IsOpen == true)
            {
                PhotonNetwork.CurrentRoom.IsOpen = false;
                //게임이 시작되면 다른 유저들은 들어오지 못하도록 막는 부분
                //PhotonNetwork.CurrentRoom.IsVisible = false;
                //로비에서 방 목록에서도 보이지 않게 하기...
            }
            if (0.0f < m_GoWaitGame) //타이머 카운트 처리
            {
                m_MgWaitGame=0.0f;  
                m_GoWaitGame -= Time.deltaTime;
                if (m_WaitTmText != null)
                {
                    m_WaitTmText.gameObject.SetActive(true);
                    m_WaitTmText.text = ((int)m_GoWaitGame).ToString();
                }
                if (m_GoWaitGame <= 0.0000001f)
                {
                    PhotonNetwork.IsMessageQueueRunning = false;
                    //백그라운드로 씬 로딩
                    SceneManager.LoadSceneAsync("LoadScene");
                }
            }
        }
    }

```
</details>

<details>
  <summary> 멀티 방만들기 및 팀선택 gif (열기/닫기)</summary>
bandicam2025-06-0121-21-03-774-ezgif.com-video-to-gif-converter.gif
</details>
ㅉ
<details>
  <summary>🔍 게임 다시하기 (열기/닫기)</summary>
</details>

```csharp
void start()
{
    //// - 각 플레이어가 Re (0=Yes, 1=No)로 투표
// - CheckRestartCondition()에서 투표 결과 평가
//   - 1명이라도 No면 투표 실패 → 초기화
//   - 전원 Yes면 RestartGame()으로 씬 재시작
// - 마스터 클라이언트는 DoRestartGame()으로 Voice, 오브젝트 등 정리 후 씬 로딩
    if (ReYesBtn != null) // 다시하기 yes버튼
        {
            ReYesBtn.onClick.AddListener(() =>
            {
                Debug.Log("예 버튼 클릭됨");

                RePlyaerHasTable["Re"] = 0; // 0이면 다시하기 선택
                PhotonNetwork.LocalPlayer.SetCustomProperties(RePlyaerHasTable);
                photonView.RPC("OnRestartVoted", RpcTarget.All, PhotonNetwork.LocalPlayer.ActorNumber, 0); // 0은 Yes
                YseNoObj.gameObject.SetActive(false);
                DegiText.gameObject.SetActive(true);
            });
        }
    if (ReNoBtn != null) 다시하기 거절버전
        {
            ReNoBtn.onClick.AddListener(() =>
            {
                Debug.Log("아니요 버튼 클릭됨");

                RePlyaerHasTable["Re"] = 1; // 1이면 다시하기 거부

                PhotonNetwork.LocalPlayer.SetCustomProperties(RePlyaerHasTable);
                photonView.RPC("OnRestartVoted", RpcTarget.All, PhotonNetwork.LocalPlayer.ActorNumber, 1); // 1은 nO

                YseNoObj.gameObject.SetActive(true);
                DegiText.gameObject.SetActive(false);

            });
        }
}
  public void CheckAgain()
  {
      pv.RPC("ShowResetImage", RpcTarget.All, true); //죽거나 미션 실패할경우 특정 다시하기 이미지띄우기
  }
  [PunRPC]
  void OnRestartVoted(int actorNumber, int vote) // 
  {
      // actorNumber로 Player 찾기
      Player player = PhotonNetwork.CurrentRoom.GetPlayer(actorNumber);
      CheckRestartCondition();
      // 직접 리스트 만들어서 투표 체크 로직 구현 가능
  }

  // 모든 플레이어의 선택을 확인해서 한 명이라도 "No"면 다시하기 불가능
  private void CheckRestartCondition()
  {
      int totalPlayers = PhotonNetwork.PlayerList.Length;
      int yesCount = 0;

      foreach (Player player in PhotonNetwork.PlayerList)
      {
          if (player.CustomProperties.ContainsKey("Re"))
          {
              int reValue = (int)player.CustomProperties["Re"];

              if (reValue == 1)
              {
                  // 누군가 No 눌렀을 때
                  string msg = "\n<color=#00ff00>" + player.NickName + "동료가 서랜을 누르지 못했어  다시 투표하고 싶으면 /gg를 치세요 </color>";
                  pv.RPC("LogMsg", RpcTarget.AllBuffered, msg, true);
                  pv.RPC("ShowResetImage", RpcTarget.All, false);
                  pv.RPC("ResetAllVotes", RpcTarget.AllBuffered); // ✅ 모든 사람 투표 초기화
                  return;
              }
              else if (reValue == 0)
              {
                  yesCount++;
              }
          }
      }
      if (yesCount == totalPlayers)
      {
          string msg = "\n<color=#00ff00>모든 플레이어가 Yes 선택 → 게임 다시 시작</color>";
          photonView.RPC("RestartGame", RpcTarget.All);
          ResetImage.SetActive(false);
      }
  }
  // 모든 플레이어의 선택을 확인해서 한 명이라도 "No"면 다시하기 불가능

  [PunRPC]
  private void RestartGame()
  {
      if (m_GameRound == GameRound.Round_1)
      {
          PhotonNetwork.LoadLevel("Round_1");
      }
      else if (m_GameRound == GameRound.Round_2)
      {
          PhotonNetwork.LoadLevel("Round_2");
      }
  }
  [PunRPC]
  void ResetAllVotes()
  {
      ResetVoteStatus();
  }

  // ✅ 마스터만 씬 리로드 실행
  [PunRPC]
  void DoRestartGame()
  {
      if (PhotonNetwork.IsMasterClient)
      {
          StartCoroutine(RestartGameCoroutine());
      }
  }

  IEnumerator RestartGameCoroutine()
  {
      //  1. 네트워크 이벤트 중단 (중요: Voice Join 시도를 막기 위해)
      PhotonNetwork.IsMessageQueueRunning = false;

      //  2. VoiceClient 명시적 종료 및 파괴
      if (PunVoiceClient.Instance != null)
      {
          if (PunVoiceClient.Instance.Client.IsConnected)
          {
              Debug.Log("[Voice] Disconnecting...");
              PunVoiceClient.Instance.Disconnect();  // 연결 끊기
          }

          // 🔥 이거 안 하면 문제가 계속 남음
          Destroy(PunVoiceClient.Instance.gameObject);  // 오브젝트 자체 파괴
      }

      //  3. 네트워크 오브젝트 모두 제거
      PhotonNetwork.DestroyAll();

      //  4. 딜레이 (정리 시간 확보)
      yield return new WaitForSeconds(0.5f);

      //  5. 메시지 큐 재시작
      PhotonNetwork.IsMessageQueueRunning = true;

      //  6. 씬 로딩
      PhotonNetwork.LoadLevel("Round_1");
  }
  ```
<details>
  <summary>🔍 채팅 기능 코드 (열기/닫기)</summary>

```csharp
//--- 채팅 관련 변수
public InputField InputFdChat; // 채팅입력정보 
public GameObject InputLogMsg; // 입력시 출력
public static bool bEnter = false;
//--- 채팅 관련 변수

void Update()
{
    if (Input.GetKeyDown(KeyCode.Return) == true)
    { //엔터키를 누르면 채팅 필드 활성화
        bEnter = !bEnter;

        if (bEnter == true)
        {
            InputFdChat.gameObject.SetActive(true);
            InputFdChat.ActivateInputField();   //<-- 키보드 커서 입력 상자 쪽으로 가게 만들어 줌
        }
        else
        {
            InputFdChat.gameObject.SetActive(false);
            if (string.IsNullOrEmpty(InputFdChat.text.Trim()) == false)
            {
                BroadcastingChat();
            }
        }
    }//if(Input.GetKeyDown(KeyCode.Return) == true)
        //채팅변수
}
 List<string> m_MsgList = new List<string>();
 // info.sender 는 메시지를 보낸 플레이의 정보를 담고 있다.
 //if (info.Sender.IsLocal == true) // 내가 보낸 메시지인 경우를 의미함
 [PunRPC]
 void LogMsg(string msg, bool isChatMsg, PhotonMessageInfo info)
 {
     //로컬에서 내가 보낸 메시지인 경우만
     //info.Sender.IsLocal == true //로컬 에서 보낸 메시지 임
     //info.Sender.ActorNumber == PhotonNetwork.LocalPlayer.ActorNumber(IsMine의 고유번호)
     if (info.Sender.IsLocal == true && isChatMsg == true)
     {
         //방장이 말을 한 경우는 "#00ffff" 로 들어 오니까 방장이 한 말은 자신도 그냥 하늘색으로 보일 것임
         msg = msg.Replace("#ffffff", "#ffff00");
     }  
     m_MsgList.Add(msg);
     if (9 < m_MsgList.Count)
         m_MsgList.RemoveAt(0);

     txtLogMsg.text = "";
     for (int i = 0; i < m_MsgList.Count; i++)
     {
         txtLogMsg.text += m_MsgList[i];
     }
 }
```
</details>

<details>
  <summary>🔍 박스 던지기 (열기/닫기)</summary>

```csharp 
public void BoxOnOff() // 업데이트문 pv.ismine로 확인
{  // 박스 근처에 있을 때 잡기기능

    if (Input.GetKeyDown(KeyCode.E))
    {
        if (!isBoxDown)
            TryPickUpBox();
        else
            TryDropBox();
    }

    if (Input.GetMouseButtonDown(0) && isBoxDown)
    {
        isCharging = true;
        isThrowing = true;
        a_CamCtrl.TakeBox = true; // 카메라 스크립트에 접근하여 줌인 시작
        TagetImg.SetActive(true);

        chargeStartTime = Time.time;
    }

    if (Input.GetMouseButtonUp(0) && isCharging)
    {
        TagetImg.SetActive(false);

        isCharging = false;
        isThrowing = false;
        a_CamCtrl.TakeBox = false; //카메라 스크립트에 접근하여 줌 아웃시작

        TryThrowBox();
    }
}

private void TryPickUpBox()
{
// E 키를 누르면:
// 1. 반경 0.5m 안에 있는 박스를 탐색
// 2. 태그가 "Box"인 오브젝트만 필터링
// 3. PhotonView가 있으면 소유권 가져옴
// 4. RPC로 모든 플레이어에게 들기 명령 (손 위치로 이동)
// 5. 들고 있는 상태로 내부 변수 갱신
// 번외 튜토리얼이면 박스 줍기 완료 보고
    Collider[] hits = Physics.OverlapSphere(transform.position, 0.5f); // 근처의 콜라이도 확인하기
    foreach (var hit in hits)
    {
        if (hit.CompareTag("Box"))
        {
            PhotonView pv = hit.GetComponent<PhotonView>();
            if (pv == null) continue;

            // 소유권 이전
            if (!pv.IsMine)
                pv.TransferOwnership(PhotonNetwork.LocalPlayer);

            Vector3 handPos = handAnchor.position;
            Quaternion handRot = handAnchor.rotation;

            pv.RPC("PickUp", RpcTarget.AllViaServer, handPos, handRot, PhotonView.Get(this).ViewID);

            currentBoxPhotonView = pv;
            heldBox = hit.gameObject;
            isBoxDown = true;
            //튜토리얼만 따로
            if (TutorialManager.Instance.currentStep == TutorialStepType.Box)
            {
                TutorialManager.Instance.photonView.RPC(
                    "NotifyStepDone_Single",
                    RpcTarget.MasterClient,
                    PhotonNetwork.LocalPlayer.ActorNumber,
                    (int)TutorialStepType.Box
                );
            }
            break;
        }
    }
}

private void TryDropBox()
{
// 박스를 내려놓는 처리
// 1. 현재 박스를 들고 있다면,
// 2. 플레이어 앞에 놓을 위치 계산
// 3. RPC로 모든 클라이언트에 박스 놓기 명령
// 4. 내부 상태 초기화 (박스 정보 비우기)
    if (currentBoxPhotonView != null)
    {
        Vector3 dropPos = transform.position + transform.forward * 0.5f;
        currentBoxPhotonView.RPC("Drop", RpcTarget.AllViaServer, dropPos);

        currentBoxPhotonView = null;
        heldBox = null;
        isBoxDown = false;
    }
}

private void TryThrowBox()
{
// 박스 던지기 로직
// 1. 마우스를 누른 시간으로 힘 계산
// 2. 플레이어 정면 + 약간 위 방향으로 벡터 설정
// 3. UI에 던질 방향 표시 (화살표)
// 4. 박스에 던지기 RPC 호출 (힘 벡터 전달)
// 5. 들고 있던 박스 상태 초기화
    if (currentBoxPhotonView != null)
    {
        float chargeTime = Time.time - chargeStartTime;
        float throwStrength = Mathf.Clamp(chargeTime * 10f, minForce, maxForce);

        // ✅ 플레이어 정면 방향으로 던지도록 수정
        Vector3 forward = new Vector3(transform.forward.x, 0f, transform.forward.z).normalized;
        forward.y = 0.2f;

        // 화살표 UI도 해당 방향에 맞춰 표시
        Vector3 targetWorldPos = transform.position + Vector3.up * 1f + forward * 1.5f;
        Vector3 screenPos = Camera.main.WorldToScreenPoint(targetWorldPos);

        TagetImg.gameObject.SetActive(true);
        TagetImg.transform.position = screenPos;

        float angle = Mathf.Atan2(forward.x, forward.z) * Mathf.Rad2Deg;
        TagetImg.transform.rotation = Quaternion.Euler(0, 0, -angle);

        // 던지기
        currentBoxPhotonView.RPC("Throw", RpcTarget.AllViaServer, forward * throwStrength);

        currentBoxPhotonView = null;
        heldBox = null;
        isBoxDown = false;
    }
}

void CheckNear()
{
// 텍스트 추출
    Collider[] hits = Physics.OverlapSphere(transform.position, 0.4f);
    foreach (var hit in hits)
    {
        if (hit.CompareTag("Box"))
        {
            MassgeText.text = "E키로 박스들기";
            return;
        }
        if (hit.CompareTag("Laver"))
        {
            MassgeText.text = "E키로 레버 당기기";
            return;
        }
    }

    MassgeText.text = " "; // 근처에 없으면 지움
}
private void TryPickDownBox()   
{
    MassgeText.text = "";
    if (currentBoxPhotonView != null)
    {
        // 박스를 놓을 위치 설정 (플레이어 앞)
        Vector3 dropPosition = transform.position + transform.forward * 0.5f;  // 앞쪽으로 2유닛

        // 박스를 놓는 RPC 호출
        currentBoxPhotonView.RPC("Drop", RpcTarget.AllViaServer, dropPosition);

        // 박스를 놓은 후 상태 초기화
        currentBoxPhotonView = null;
        isBoxDown = false;
    }

}
void RotateToCameraForward()
{
    // 카메라 정면 방향을 향해 부드럽게 회전
// 수평 방향만 고려해서 캐릭터가 기울지 않게 함
// Slerp로 자연스럽게 회전 애니메이션 적용
    Vector3 lookDir = cameraTransform.forward;
    lookDir.y = 0f;  // 수평 방향만

    if (lookDir.sqrMagnitude > 0.01f)
    {
        Quaternion targetRot = Quaternion.LookRotation(lookDir);
        transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, Time.deltaTime * 10f);  // 부드럽게 회전
    }
}

// 여기부터는 박스 스크립트
    [PunRPC]
    public void PickUp(Vector3 handPosition, Quaternion handRotation, int actorID)
    {
// 박스를 들었을 때 실행되는 RPC
// 1. 소유권이 없으면 요청자에게 이전
// 2. Rigidbody 물리 제거 (Kinematic + Trigger) -> 하지않으면 밀려나감
// 3. 손 위치에 박스를 붙이고 회전/위치 초기화
// 4. "들고 있음" 상태로 전환 및 손 기준 정보 저장
        if (!photonView.IsMine)
            photonView.TransferOwnership(actorID);

        rb.isKinematic = true;
        boxCollider.isTrigger = true;

        Transform handAnchor = PhotonView.Find(actorID).GetComponent<PlayerCtrl>().handAnchor;

        transform.SetParent(handAnchor);
        transform.localPosition = Vector3.zero;
        transform.localRotation = Quaternion.identity;

        isFloating = true;

        isHeld = true;
        holderHandAnchor = handAnchor; // 🔹 기억해놓기
    }
    [PunRPC]
    public void Drop(Vector3 dropPos)
    {
        //  박스를 내려놓을 때 호출되는 RPC
// 1. 손과의 부모 관계 해제
// 2. 플레이어가 지정한 위치로 이동
// 3. 물리 속성 복구 (Kinematic off, Trigger off)
// 4. 상태값 초기화 (들고 있는 상태 해제)
        transform.SetParent(null);
        transform.position = dropPos;

        rb.isKinematic = false;
        boxCollider.isTrigger = false;

        isHeld = false;
        holderHandAnchor = null;
    }
    [PunRPC]
    public void Throw(Vector3 force)
    {
// 박스를 던질 때 호출되는 RPC
// 1. 손에서 분리하고 앞쪽으로 약간 이동
// 2. 회전 초기화
// 3. Rigidbody 물리 적용 (속도 초기화)
// 4. 충돌체 활성화 → 충돌 가능 상태
// 5. 힘을 가해 박스를 날림 (충전 시간 기반)
// 6. 0.5초 후 다시 Trigger 켜기 (재줍기 가능 상태로)
        transform.SetParent(null);

        // 1. 던지기 직전, 플레이어에서 약간 앞 위치로 이동
        transform.position += force.normalized * 0.5f;

        // 2. 회전 초기화 (필요시)
        transform.rotation = Quaternion.identity;

        // 3. 물리 준비
        rb.isKinematic = false;
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;

        // 4. 충돌체 켜기 (이제 겹치지 않음)
        boxCollider.isTrigger = false;

        // 5. 힘 적용
        rb.AddForce(force, ForceMode.Impulse);

        isFloating = false;

        StartCoroutine(EnableTriggerAfterDelay(0.5f));

    }
    //플레임을 늦춰서 다시 키기
    IEnumerator EnableTriggerAfterDelay(float delay)
    {
        yield return new WaitForSeconds(delay);

        if (!isHeld)
            boxCollider.isTrigger = true; // 다시 줍기 가능 상태로 전환
    }
    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Tutorial"))
        {
            Debug.Log("타겟 명중!");

            PhotonView tutorialPV = TutorialManager.Instance.photonView;
            tutorialPV.RPC("NotifyStepDone_Single", RpcTarget.MasterClient, PhotonNetwork.LocalPlayer.ActorNumber, (int)TutorialStepType.BoxThrow);
        }
        if (other.CompareTag("Water") && !isFloating)
        {
            Debug.Log("박스빠짐!");
            // 1. 회전 정지
            rb.angularVelocity = Vector3.zero;
            boxCollider.isTrigger = false;
            rb.isKinematic = true;
            // 2. 회전 고정 (선택적)
            rb.constraints |= RigidbodyConstraints.FreezeRotation;
            StartCoroutine(FloatThenSink());
        }
    }
    IEnumerator FloatThenSink()
    {
        // 박스가 물에 빠졌을 때 나타나는 효과 코드
        isFloating = true;

        // 1. 물리 멈춤 및 회전 제한
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;
        rb.constraints |= RigidbodyConstraints.FreezeRotation;

        // 2. 위치 살짝 내리기
        float floatYOffset = -0.5f;
        Vector3 floatBasePos = transform.position + new Vector3(0, floatYOffset, 0);
        transform.position = floatBasePos;

        float timer = 0f;
        Quaternion targetRotation = Quaternion.Euler(0f, transform.rotation.eulerAngles.y, 0f);

        // 3. 떠 있는 동안 흔들리고 회전 보정
        while (timer < floatDuration)
        {
            timer += Time.deltaTime;

            float yOffset = Mathf.Sin(Time.time * bobSpeed) * bobHeight;
            Vector3 newPos = new Vector3(
                floatBasePos.x,
                floatBasePos.y + yOffset,
                floatBasePos.z
            );
            transform.position = newPos;

            transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, Time.deltaTime * rotationSmooth);

            yield return null;
        }

        // 4. 가라앉기 시작
        float sinkTimer = 0f;
        while (sinkTimer < sinkDuration)
        {
            sinkTimer += Time.deltaTime;

            transform.position -= new Vector3(0, sinkSpeed * Time.deltaTime, 0);

            yield return null;
        }

        // 5. 삭제
        Destroy(gameObject);
    }

    void Update()
    {
        if (PlayerOnTop())
        {
            GameObject player = GameObject.FindGameObjectWithTag("Player");

            Vector3 targetPos = new Vector3(
                transform.position.x,
                player.transform.position.y, // 높이는 그대로
                transform.position.z
            );

            // 자연스럽게 보정
            player.transform.position = Vector3.Lerp(
                player.transform.position,
                targetPos,
                Time.deltaTime * 1.5f // 속도 조절 가능
            );
        }
    }
    public void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info)
    {
        if (isHeld) return; // ❗ 박스를 들고 있으면 위치 동기화 안 함

        if (stream.IsWriting)
        {
            stream.SendNext(transform.position);
            stream.SendNext(transform.rotation);
        }
        else
        {
            transform.position = (Vector3)stream.ReceiveNext();
            transform.rotation = (Quaternion)stream.ReceiveNext();
        }
    }
```
</details>

<details>
  <summary>1라운드 트랩 코드 (열기/닫기)</summary>
</details>
