using ListingUtil;
using TMPro;
using UdonSharp;
using UnityEngine;
using UnityEngine.Serialization;
using UnityEngine.UI;
using VRC.SDKBase;
using VRC.Udon;

public enum eObangState
{
    none,
    Press_To_Start,
    Watch_Description,
    Pick_Frame,
    // Pick_BackGround,
    Pick_Front,
    Take_4Cuts,
    Decide_to_ReTake_4Cuts,
    Printing,
    Finish_print,
}

public enum eFrame
{
    // none, 
    FR겨울패턴,
    FR핑크클라우드,
    FR왁타랜드,
    FR심플블랙,
    Max,
}

public enum eFront
{
    // none, 
    F_아이네1, F_아이네2, F_아이네3, F_아이네4, 
    F_징버거1, F_징버거2, F_징버거3, F_징버거4,
    F_릴파1, F_릴파2, F_릴파3, F_릴파4,
    F_주르르1, F_주르르2,F_주르르3,F_주르르4,
    F_고세구1, F_고세구2, F_고세구3, F_고세구4,
    F_비챤1,F_비챤2,F_비챤3,F_비챤4,
    F_우왁굳1, F_해루석1,
    Max,
}

public enum eBackGround
{
    // none,
    BG_겨울패턴,
    BG_핑크클라우드,
    BG_왁타랜드,
    BG_심플블랙,
    Max,
}

public class WTObang4CutManager : UdonSharpBehaviour
{
    // 사진 찍는거 관련 : WTCameraManager
    // 오뱅네컷 스테이트 등 진행 상황 관련 : WTObang4CutManager

    public AudioSource obangAudio;
    public AudioClip[] _Clips;
    
    public WTObangCameraManager CameraManager;

    [SerializeField] private TextMeshProUGUI ownerText;
    
    // 결과물 인쇄 후 원래 화면으로 돌아오기 위해 기다리는 시간
    [SerializeField] private byte resetDelay; 
    
    // 현재 프레임 , 현재 인덱스() , 현재 스테이트
    [SerializeField, UdonSynced(), FieldChangeCallback(nameof(CurFrame))] private eFrame _curFrame = eFrame.FR겨울패턴;

    public eFrame CurFrame
    {
        get => _curFrame;
        set
        {
            _curFrame = value;
            // OnCurFrameChanged()
            Debug.Log($"CurFrame = {CurFrame}");
            OnCurFrameChanged(CurFrame);
        }
    }

    [SerializeField, UdonSynced(), FieldChangeCallback(nameof(CurIndex))]private byte _curIndex;

    public byte CurIndex
    {
        get => _curIndex;
        set
        {
            _curIndex = value;
            OnIndexChanged(CurIndex);
        }
    }

    public eObangState curState;
    // public eBackGround curBG;

    [UdonSynced(), FieldChangeCallback(nameof(CurFront1)), SerializeField] private eFront _curFront1;

    public eFront CurFront1
    {
        get => _curFront1;
        set { _curFront1 = value; OnCurFront1Changed(CurFront1); }
    }

    [UdonSynced(), FieldChangeCallback(nameof(CurFront2)), SerializeField] private  eFront _curFront2;
    public eFront CurFront2
    {
        get => _curFront2;
        set { _curFront2 = value; OnCurFront2Changed(CurFront2); }
    }
    
    [UdonSynced(), FieldChangeCallback(nameof(CurFront3)), SerializeField] private eFront _curFront3;
    public eFront CurFront3
    {
        get => _curFront3;
        set { _curFront3 = value; OnCurFront3Changed(CurFront3); }
    }
    
    [UdonSynced(), FieldChangeCallback(nameof(CurFront4)), SerializeField] private eFront _curFront4;
    public eFront CurFront4
    {
        get => _curFront4;
        set { _curFront4 = value; OnCurFront4Changed(CurFront4); }
    }
    
    public GameObject[] stateScreens;
    public GameObject[] reAskButtonObjs;
    public GameObject reAskWindow;
    // A번 사진을 다시 찍겠습니까?의 A를 담당
    public TextMeshProUGUI numberText; 
    
    
    // 오뱅네컷을 몇 번 이용했는지에 관한 인덱스
    public int _curObang4CutIndex;
    
    private int limitRTs; // 결과물용 렌더 텍스쳐의 최대 개수
    public Camera resultCam; public RenderTexture[] resultRTs;
    [SerializeField] private Material testMat;
   
    public RawImage leftBigFrameImg; public GameObject resultObject;
    public Material[] resultMaterials;
   
    [Header("Textures")]public Texture[] frameImgs, frontImgs;
    public GameObject[] leftFrontObjs, resultFrontObjs, screenFrontObjs;
    public RawImage[] leftFrontImgs, resultFrontImgs, screenFrontImgs;
    // public GameObject[] frontObjs;
    public GameObject[] bg1Objs, bg2Objs, bg3Objs, bg4Objs;
    
    
     private bool didSelectFrame = false;
     // private bool didSelectBackGround = false;
     private bool didSelectFront = false;
     
     [SerializeField, UdonSynced() , FieldChangeCallback(nameof(CurFrontIndex))] private int _curFrontIndex = -1;

     public int CurFrontIndex
     {
         get => _curFrontIndex;
         set
         {
             _curFrontIndex = value;
             // OnCurFrontIndexChanged();
             OnCurFrontIndexChanged(CurFrontIndex);
         }
     }

     #region 변수 추가 by 끽잉
    public WTSystemManager sysManager;
    #endregion
    void Start()
    {
        _curObang4CutIndex = 0;
        ResetState();
        ResetObang4CutIndex();
        CloseReTakePhoto();
        limitRTs = resultRTs.Length; // 끽잉 : int로 변경
        ResetFrame();
        ResetFront();
        // ResetBackGround();
        
    }

    public void OnIndexChanged(byte curIndex)
    {
        CheckCurIndex(curIndex);
        obangAudio.PlayOneShot(_Clips[0]);
    }

    #region 백그라운드 선택 관련 함수


    // public void ResetBackGround()
    // {
    //     didSelectBackGround = false;
    //     curBG = eBackGround.none;
    // }

    // public void CheckSelectBackGround()
    // {
    //     if(curBG.Equals(eBackGround.none))
    //     {
    //         // 백그라운드 선택 안함! 다음 단계로 넘어갈 수 없습니다.
    //         didSelectBackGround = false;
    //         UdonLogUtil.PrintErr($"백그라운드 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
    //     }
    //     else
    //     {
    //         didSelectBackGround = true;
    //         UdonLogUtil.Print($"백그라운드 선택 완료, 다음 단계로 넘어갑니다.");
    //     }
    // }

    // public void SelectBG1()
    // {
    //     curBG = eBackGround.BG_겨울패턴;
    //     foreach (var o in bgObjs)
    //     {
    //         if (o != null)
    //         {
    //             o.SetActive(false);
    //         }
    //     }
    //     bgObjs[0].SetActive(true);
    //     
    //     
    //     // leftBigFrameImg.texture = frameImgs[0];
    //     // 왼쪽 큰 프레임 관련
    //     
    //     //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[0];
    //     //didSelectBackGround = true;
    // }
    // public void SelectBG2()
    // {
    //     curBG = eBackGround.BG_핑크클라우드;
    //     foreach (var o in bgObjs)
    //     {
    //         if (o != null)
    //         {
    //             o.SetActive(false);
    //         }
    //     }
    //     bgObjs[1].SetActive(true);
    //     
    //     // leftBigFrameImg.texture = frameImgs[1];
    //
    //     //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[1];
    //
    //     //didSelectBackGround = true;
    // }
    // public void SelectBG3()
    // {
    //     curBG = eBackGround.BG_왁타랜드;
    //      foreach (var o in bgObjs)
    //     {
    //         if (o != null)
    //         {
    //             o.SetActive(false);
    //         }
    //     }
    //     bgObjs[2].SetActive(true);
    //     // leftBigFrameImg.texture = frameImgs[2];
    //
    //     //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[2];
    //
    //     //didSelectBackGround = true;
    // }
    // public void SelectBG4()
    // {
    //     curBG = eBackGround.BG_심플블랙;
    //      foreach (var o in bgObjs)
    //     {
    //         if (o != null)
    //         {
    //             o.SetActive(false);
    //         }
    //     }
    //     bgObjs[3].SetActive(true);
    //     // leftBigFrameImg.texture = frameImgs[3];
    //
    //     //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[3];
    //
    //     //didSelectBackGround = true;
    // }
    
  
    
    #endregion
    
    #region 프레임ㅇㅇ 선택 관련 함수


    public void ResetFrame()
    {
        if (curState != eObangState.Finish_print && curState != eObangState.Printing)
        {
            didSelectFrame = false;
            CurFrame = eFrame.Max;
            RequestSerialization();
        }
        
    }

    public void CheckSelectFrame()
    {
        if(CurFrame.Equals(eFrame.Max))
        {
            // 프레임 선택 안함! 다음 단계로 넘어갈 수 없습니다.
            didSelectFrame = false;
            UdonLogUtil.PrintErr($"프레임 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
        }
        else
        {
            didSelectFrame = true;
            UdonLogUtil.Print($"프레임 선택 완료, 다음 단계로 넘어갑니다.");
        }
    }

    public void SelectFrame1()
    {
        if (!Networking.IsOwner(gameObject)) return;
        
        CurFrame = eFrame.FR겨울패턴;
        //leftBigFrameImg.texture = frameImgs[0];
        //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[0];
        RequestSerialization();
    }
    public void SelectFrame2()
    {
        if (!Networking.IsOwner(gameObject)) return;
        
        CurFrame = eFrame.FR핑크클라우드;
        //leftBigFrameImg.texture = frameImgs[1];
        //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[1];
        RequestSerialization();
    }
    public void SelectFrame3()
    { 
        if (!Networking.IsOwner(gameObject)) return;
      
        CurFrame = eFrame.FR왁타랜드;
        //leftBigFrameImg.texture = frameImgs[2];
        //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[2];
        RequestSerialization();
    }
    public void SelectFrame4()
    {
         if (!Networking.IsOwner(gameObject)) return;
         
         CurFrame = eFrame.FR심플블랙;
        //leftBigFrameImg.texture = frameImgs[3];
        //resultObject.GetComponent<MeshRenderer>().material = resultMaterials[3];
        RequestSerialization();
    }

    public void OnCurFrameChanged(eFrame curFrame)
    {
        leftBigFrameImg.texture = frameImgs[(int)curFrame];
        resultObject.GetComponent<MeshRenderer>().material = resultMaterials[(int)curFrame];
    }

    #endregion

    #region 프론트 선택 관련 함수
    public void ResetFront()
    {
        if (curState != eObangState.Printing && curState != eObangState.Finish_print)
        {
            didSelectFront = false;
            CurFront1 = eFront.Max; CurFront2 = eFront.Max;
            CurFront3 = eFront.Max; CurFront4 = eFront.Max;
            CurFrontIndex = 0;
            foreach (var o in leftFrontImgs)
            {
                o.gameObject.SetActive(false);
            }
            RequestSerialization();

        }
    }

    public void CheckSelectFront()
    {
        if (CurFront1.Equals(eFront.Max))
        {
            // 프레임 선택 안함! 다음 단계로 넘어갈 수 없습니다.
            didSelectFront = false;
            UdonLogUtil.PrintErr($"프론트1 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
            // MinusFrontState();
        }
        else if (CurFront2.Equals(eFront.Max))
        {
            UdonLogUtil.PrintErr($"프론트2 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
            didSelectFront = false;
            // MinusFrontState();
        }
        else if (CurFront3.Equals(eFront.Max))
        {
            UdonLogUtil.PrintErr($"프론트3 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
            didSelectFront = false;
            // MinusFrontState();
        }
        else if (CurFront4.Equals(eFront.Max))
        {
            UdonLogUtil.PrintErr($"프론트4 선택 안함! 다음 단계로 넘어갈 수 없습니다.");
            didSelectFront = false;
            // MinusFrontState();
        }
        else
        {
            didSelectFront = true;
            UdonLogUtil.Print($"프론트 선택 완료, 다음 단계로 넘어갑니다.");
        }

        
    }

    public void PlusFrontState()
    {
        if (!Networking.IsOwner(gameObject)) return;
        
        CurFrontIndex++;
        //OnCurFrontIndexChanged(CurFrontIndex);
        RequestSerialization();
    }

    public void MinusFrontState()
    {
        if (!Networking.IsOwner(gameObject)) return;
        
        CurFrontIndex--;
        //OnCurFrontIndexChanged(CurFrontIndex);
        RequestSerialization();

    }

    public void MinusState_Front()
    {
        MinusState();
        ResetFront();
    }
    

    public void OnCurFrontIndexChanged(int curFrontIndex)
    {
        switch (curFrontIndex)
        {
            case -1 : 
                // 이전 화면으로 돌아간다
                
                // MinusState();
                // ResetFront();
                break;
            case 0: //
                // 
                // _curFront1 =
                // CheckSelectFront();
                // 1번째 단계로 돌아옴
                // UdonLogUtil.Print($"0번째 단계 끝");
                break;
            
            case 1 : 
                //
                // 2번째 단계로 돌아옴 / 2번째 단계로 넘어감
                //_curFront1
                
                
                UdonLogUtil.Print($"1번째 단계 끝");
                break;
            
            case 2 : 
                //
                // 3번째 단계로 돌아옴 / 3번째 단계로 넘어감
                UdonLogUtil.Print($"2번째 단계 끝");
                break;
            
            case 3 : 
                //
                // 4번째 단계로 돌아옴 / 4번째 단계로 넘어감
                UdonLogUtil.Print($"3번째 단계 끝");
                break;
            
            case 4 :
                
                // PlusState();
                // ResetFront();
                UdonLogUtil.Print($"4번째 단계 끝");
                break;
            
            
            
            
        }
        
    }

    public void OnCurFront1Changed(eFront curFront1)
    {
        leftFrontObjs[0].SetActive(true);
        leftFrontImgs[0].texture = frontImgs[(int)curFront1];
        resultFrontObjs[0].SetActive(true);
        resultFrontImgs[0].texture = frontImgs[(int)curFront1];
        
        screenFrontImgs[0].texture = frontImgs[(int)curFront1];
    }
    public void OnCurFront2Changed(eFront curFront2)
    {
        leftFrontObjs[1].SetActive(true);
        leftFrontImgs[1].texture = frontImgs[(int)curFront2];
        resultFrontObjs[1].SetActive(true);
        resultFrontImgs[1].texture = frontImgs[(int)curFront2];
        
        screenFrontImgs[1].texture = frontImgs[(int)curFront2];
    }
    public void OnCurFront3Changed(eFront curFront3)
    {
        leftFrontObjs[2].SetActive(true);
        leftFrontImgs[2].texture = frontImgs[(int)curFront3];
        resultFrontObjs[2].SetActive(true);
        resultFrontImgs[2].texture = frontImgs[(int)curFront3];
        
        screenFrontImgs[2].texture = frontImgs[(int)curFront3];
    }
    public void OnCurFront4Changed(eFront curFront4)
    {
        leftFrontObjs[3].SetActive(true);
        leftFrontImgs[3].texture = frontImgs[(int)curFront4];
        resultFrontObjs[3].SetActive(true);
        resultFrontImgs[3].texture = frontImgs[(int)curFront4];
        
        screenFrontImgs[3].texture = frontImgs[(int)curFront4];
    }


    /**
     * 1. _curFront1 // FrontIndex 따로, 
     */
    
    // todo : leftFrontObjs는 샘플용 왼쪽 큰 프레임이다. 결과물에도 frontObjs 세팅 추가해줘야 한다!!
    public void SelectFront1()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_아이네1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_아이네1];
                PlusFrontState();
                //todo: 여기서 동기화 요청을 또 해야할까? 
                //RequestSerialization();
                break;
            
            case 1 : 
                CurFront2 = eFront.F_아이네1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_아이네1];
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 = eFront.F_아이네1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_아이네1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 = eFront.F_아이네1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_아이네1];
                PlusFrontState();
                break;
        }
        
    }
    
    
    public void SelectFront2()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_아이네2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_아이네2];
                PlusFrontState();
                break;
            
            case 1 : 
               CurFront2 = eFront.F_아이네2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_아이네2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 = eFront.F_아이네2;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_아이네2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 = eFront.F_아이네2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_아이네2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront3()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_아이네3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_아이네3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 = eFront.F_아이네3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_아이네3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 = eFront.F_아이네3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_아이네3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 = eFront.F_아이네3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_아이네3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront4()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_아이네4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_아이네4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 = eFront.F_아이네4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_아이네4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 = eFront.F_아이네4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_아이네4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 = eFront.F_아이네4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_아이네4];
                PlusFrontState();
                break;
        }
    }
   
    public void SelectFront5()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_징버거1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_징버거1];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_징버거1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_징버거1];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_징버거1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_징버거1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_징버거1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_징버거1];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront6()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_징버거2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_징버거2];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_징버거2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_징버거2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_징버거2;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_징버거2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_징버거2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_징버거2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront7()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_징버거3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_징버거3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_징버거3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_징버거3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_징버거3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_징버거3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_징버거3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_징버거3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront8()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_징버거4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_징버거4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_징버거4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_징버거4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_징버거4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_징버거4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_징버거4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_징버거4];
                PlusFrontState();
                break;
        }
    }
    
    
    public void SelectFront9()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_릴파1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_릴파1];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_릴파1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_릴파1];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_릴파1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_릴파1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_릴파1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_릴파1];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront10()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_릴파2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_릴파2];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_릴파2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_릴파2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_릴파2;
                //eftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_릴파2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_릴파2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_릴파2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront11()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_릴파3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_릴파3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_릴파3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_릴파3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_릴파3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_릴파3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_릴파3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_릴파3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront12()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_릴파4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_릴파4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_릴파4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_릴파4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_릴파4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_릴파4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_릴파4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_릴파4];
                PlusFrontState();
                break;
        }
    }
    
    public void SelectFront13()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_주르르1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_주르르1];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_주르르1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_주르르1];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_주르르1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_주르르1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_주르르1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_주르르1];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront14()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_주르르2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_주르르2];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_주르르2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_주르르2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_주르르2;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_주르르2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_주르르2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_주르르2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront15()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_주르르3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_주르르3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_주르르3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_주르르3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_주르르3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_주르르3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_주르르3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_주르르3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront16()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_주르르4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_주르르4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_주르르4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_주르르4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_주르르4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_주르르4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_주르르4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_주르르4];
                PlusFrontState();
                break;
        }
    }
    
    public void SelectFront17()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_고세구1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_고세구1];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_고세구1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_고세구1];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_고세구1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_고세구1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_고세구1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_고세구1];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront18()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_고세구2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_고세구2];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_고세구2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_고세구2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_고세구2;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_고세구2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_고세구2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_고세구2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront19()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_고세구3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_고세구3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_고세구3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_고세구3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_고세구3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_고세구3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_고세구3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_고세구3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront20()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_고세구4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_고세구4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_고세구4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_고세구4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_고세구4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_고세구4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_고세구4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_고세구4];
                PlusFrontState();
                break;
        }
    }
    
    public void SelectFront21()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_비챤1;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_비챤1];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_비챤1;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_비챤1];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_비챤1;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_비챤1];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_비챤1;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_비챤1];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront22()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_비챤2;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_비챤2];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_비챤2;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_비챤2];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_비챤2;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_비챤2];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_비챤2;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_비챤2];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront23()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_비챤3;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_비챤3];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_비챤3;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_비챤3];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_비챤3;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_비챤3];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_비챤3;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_비챤3];
                PlusFrontState();
                break;
        }
    }
    public void SelectFront24()
    {
        switch (CurFrontIndex)
        {
            case 0 : 
                CurFront1 = eFront.F_비챤4;
                //leftFrontObjs[0].SetActive(true);
                //leftFrontImgs[0].texture = frontImgs[(int)eFront.F_비챤4];
                PlusFrontState();
                break;
            
            case 1 : 
                CurFront2 =  eFront.F_비챤4;
                //leftFrontObjs[1].SetActive(true);
                //leftFrontImgs[1].texture = frontImgs[(int)eFront.F_비챤4];    
                PlusFrontState();
                break;
            
            case 2 : 
                CurFront3 =  eFront.F_비챤4;
                //leftFrontObjs[2].SetActive(true);
                //leftFrontImgs[2].texture = frontImgs[(int)eFront.F_비챤4];
                PlusFrontState();
                break;
            
            case 3 : 
                CurFront4 =  eFront.F_비챤4;
                //leftFrontObjs[3].SetActive(true);
                //leftFrontImgs[3].texture = frontImgs[(int)eFront.F_비챤4];
                PlusFrontState();
                break;
        }
    }
    #endregion
    

    #region 오뱅네컷 찍은 횟수와 관련된 함수 

    

    public void ResetObang4CutIndex()
    {
        _curObang4CutIndex = 0;
        UdonLogUtil.Print($"오뱅네컷 방문 횟수 초기화.");
        resultCam.targetTexture = resultRTs[0];
    }

    public void PlusObang4CutIndex()
    {
        // 0번 진행하고 그 다음에 1번 진행하고 이런 식으로 가는게 맞을듯.
        testMat.mainTexture = resultRTs[_curObang4CutIndex];
        sysManager.AddObaengImage(resultRTs[_curObang4CutIndex]); // 끽잉 추가
        
        _curObang4CutIndex++;
        if (_curObang4CutIndex >= limitRTs)
        {
            _curObang4CutIndex = 0;
            // 일정 개수 이상 찍으면 0번째부터 재사용.
        }
        resultCam.targetTexture = resultRTs[_curObang4CutIndex];

        UdonLogUtil.Print($"로컬 플레이어가 오뱅네컷을 {_curObang4CutIndex}회 방문했습니다.");
        
    }
    
    #endregion

    #region 오뱅네컷 한 Cycle 진행하면서 변화하는 State에 관한 함수

    


    public void CheckCurIndex(byte _curIndex)
    {
        switch (_curIndex)
        {
            case 0:
                curState = eObangState.none;
                CheckState(curState);
                CameraManager.ResetBGs();
                break;
            case 1:
                curState = eObangState.Press_To_Start;
                CheckState(curState);
                break;
            case 2:
                curState = eObangState.Watch_Description;
                CheckState(curState);
                break;
            case 3:
                curState = eObangState.Pick_Frame;
                CheckState(curState);
                break;
            // case 4:
            //     curState = eObangState.Pick_BackGround;
            //     CheckSelectFrame();
            //     CheckState(curState);
            //     break;
            case 4:
                curState = eObangState.Pick_Front;
                // CheckSelectBackGround();
                CheckState(curState);
                break;
            case 5:
                curState = eObangState.Take_4Cuts;
                CheckSelectFront();
                CheckState(curState);
                break;
            case 6:
                curState = eObangState.Decide_to_ReTake_4Cuts;
                CheckState(curState);
                break;
            case 7:
                curState = eObangState.Printing;
                CheckState(curState);
                break;
            case 8:
                curState = eObangState.Finish_print;
                CheckState(curState);
                CameraManager.ResetBGs();
                break;

        }
    }

    public void CheckState(eObangState curState)
    {
        switch (curState)
        {
            case eObangState.none:
                UdonLogUtil.PrintErr($"curState is None!");
                break;

            case eObangState.Press_To_Start:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[0].SetActive(true);
                UdonLogUtil.Print($"curState : Press_To_Start");
                // ResetFrame();
                // ResetFront();
                // ResetBackGround();
                break;

            case eObangState.Watch_Description:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[1].SetActive(true);
                UdonLogUtil.Print($"curState : Watch_Description");
                ResetFrame();
                ResetFront();
                break;

            case eObangState.Pick_Frame:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[2].SetActive(true);
                UdonLogUtil.Print($"curState : Pick_Frame");
                break;

            // case eObangState.Pick_BackGround:
            //     foreach (var o in stateScreens)
            //     {
            //         o.SetActive(false);
            //     }
            //
            //     stateScreens[3].SetActive(true);
            //     UdonLogUtil.Print($"curState : Pick_BackGround");
            //     
            //     break;
            
            case eObangState.Pick_Front :
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }
                
                stateScreens[3].SetActive(true);
                UdonLogUtil.Print($"curState : Pick_Front");
                
                break;
            
            case eObangState.Take_4Cuts:
                // if (didSelectFrame && didSelectFront)
                {
                    foreach (var o in stateScreens)
                    {
                        o.SetActive(false);
                    }

                    stateScreens[4].SetActive(true);
                    Start4Photos();
                    UdonLogUtil.Print($"curState : Take_4Cuts");
                    break;
                }
                // else
                // {
                //     // 프레임, Front, BG 중 하나 선택 안함!
                //     MinusState();
                //     UdonLogUtil.PrintErr($"Frame or Front 선택 안함, MinusState 실행됨.");
                //     break;
                // }
                

                break;

            case eObangState.Decide_to_ReTake_4Cuts:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[5].SetActive(true);
                UdonLogUtil.Print($"curState : Decide_to_ReTake_4Cuts");

                break;

            case eObangState.Printing:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[6].SetActive(true);
                AnimatingResultPrint();
                UdonLogUtil.Print($"curState : Printing");

                break;

            case eObangState.Finish_print:
                foreach (var o in stateScreens)
                {
                    o.SetActive(false);
                }

                stateScreens[7].SetActive(true);
                DelayResetState();
                //ResetFrame();
                
                // 오뱅네컷 방문 횟수를 1회 늘리고, 최종 결과물용 카메라 렌더 텍스쳐를 변환시키기 위함.
                PlusObang4CutIndex();
                UdonLogUtil.Print($"curState : Finish_print");

                break;

        }
    }

    public void RegisterOwner()
    {
        UdonNetworkUtil.TransferOwnership(gameObject);
        
        // _curIndex = 3으로
        // _curIndex = 3;
        PlusState();
        Debug.Log($"CurIndex = {CurIndex}");
        // RequestSerialization();
    }

    public void LocalPlusState()
    {
        obangAudio.PlayOneShot(_Clips[0]);
        CurIndex++;
        CheckCurIndex(CurIndex);
    }


    public void PlusState()
    {
        if (!Networking.IsOwner(gameObject)) return;
        
        //obangAudio.PlayOneShot(_Clips[0]);
        CurIndex++;
        CheckCurIndex(CurIndex);
        RequestSerialization();
        
    }

    public void MinusState()
    {
        if (!Networking.IsOwner(gameObject)) return;

        //obangAudio.PlayOneShot(_Clips[0]);
        CurIndex--;
        CheckCurIndex(CurIndex);
        RequestSerialization();
    }

    public void ResetState()
    {
        CurIndex = 1;
        CheckCurIndex(CurIndex);
        foreach (var O in reAskButtonObjs)
        {
            O.SetActive(false);
        }
        numberText.text = "";
        reAskWindow.SetActive(false);
        RequestSerialization();
    }

    public void DelayResetState()
    {
        SendCustomEventDelayedSeconds(nameof(ResetState), resetDelay);
        
    }
    
    #endregion


    public void Start4Photos()
    {
        CameraManager.StartPhoto();
    }

    public void ReAsk1()
    {
        // 1 ~ 4번 사진을 다시 찍기 위해 버튼 클릭을 할 때 실행되는 메서드.
        
        // 1. 창이 뜬다 (다른 버튼들은 비활성화 된다.)
        // 2. 다시 찍기 / 취소 버튼이 활성화 된다.
        // 3. 다시 찍기 버튼을 누르면 1번 ~ 4번 중 어떤걸 눌렀는지에 따라
        // WTObangCameraManager.OnlyTakeCamera1 ~ OnlyTakeCamera4 를 실행시키면 된다.
        reAskWindow.SetActive(true);
        foreach (var O in reAskButtonObjs)
        {
            O.SetActive(false);
        }
        reAskButtonObjs[0].SetActive(true);
        numberText.text = $"1";
    }

    public void ReAsk2()
    {        
        reAskWindow.SetActive(true);
        foreach (var O in reAskButtonObjs)
        {
            O.SetActive(false);
        }
        reAskButtonObjs[1].SetActive(true);
        numberText.text = $"2";
    }
    
    public void ReAsk3()
    { 
        reAskWindow.SetActive(true);
        foreach (var O in reAskButtonObjs)
        {
            O.SetActive(false);
        }
        reAskButtonObjs[2].SetActive(true);
        numberText.text = $"3";
    }
    
    public void ReAsk4()
    {      
        reAskWindow.SetActive(true);
        foreach (var O in reAskButtonObjs)
        {
            O.SetActive(false);
        }
        reAskButtonObjs[3].SetActive(true);
        numberText.text = $"4";
    }
    
    
    
    public void CloseReTakePhoto()
    {
        reAskWindow.SetActive(false);
    }

    public void ReTakePhoto1() => ReTakePhoto(0);
    public void ReTakePhoto2() => ReTakePhoto(1);
    public void ReTakePhoto3() => ReTakePhoto(2);
    public void ReTakePhoto4() => ReTakePhoto(3);

    public void ReTakePhoto(byte photoIndex)
    {
        switch (photoIndex)
        {
            case 0 : CameraManager.OnlyTakeCamera1();
                CloseReTakePhoto();
                UdonLogUtil.Print($"첫번째 사진을 다시 찍습니다.");
                break;
            
            case 1 : CameraManager.OnlyTakeCamera2();
                CloseReTakePhoto();
                UdonLogUtil.Print($"두번째 사진을 다시 찍습니다.");
                break;
            
            case 2 : CameraManager.OnlyTakeCamera3();
                CloseReTakePhoto();
                UdonLogUtil.Print($"세번째 사진을 다시 찍습니다.");
                break;
            
            case 3 : CameraManager.OnlyTakeCamera4();
                CloseReTakePhoto();
                UdonLogUtil.Print($"네번째 사진을 다시 찍습니다.");
                break;
        }
    }

    // 카메라 매니저에서 결과물 애니메이션 실행 및 콜라이더 켜지게 한다.
    public void AnimatingResultPrint()
    {
        CameraManager.PrintPhotoAnim();
        SendCustomEventDelayedSeconds(nameof(PlusState), CameraManager.PhotoDelay + 1);
    }
    


}
위 코드를 사용하니 1회 루프를 완료했을 때 
로컬 플레이어가 오뱅네컷을 2회 방문했습니다. 가 떴어. 즉 2번 루프된거지.
이 코드 오류를 수정해주고 최적화해줘.
