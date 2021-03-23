

# SmartKey Document
####  위 문서는 fa-app에서 fa들이 현재 고객의 예약이 없는 차량을 관리하고 제어할 수 있는 스마트키에관한 문서입니다. 

####  해당 코드는 스마트키를 보여주는 SmartKeyView.js와 스마트키의 동작을 위한 `Server 통신` 과 `BLE 통신`이 이루어지는 ObjectDetailScreen.js로 이루어져 있으며, React의 Context API를 사용하여 스마트키의 동작을 위한 데이터를 관리하고 있습니다.
## function summary
|Screen / View|func name | content
|--|--|--|
|-|server 통신|-|
| objectDetailScreen.js |getTerminalsInfo()|objectUuid로 사용자가 선택 한 차량 단말기의 deviceId와 supplier를 가져오는 api.|
|objectDetailScreen.js|controlSmartKey()|사용자가 선택한controlType을 받아 SmartKeyControl api를 호출하고, response를 반환하는 함수. response의 통신이 정상적이지 않을 시 controlSmartKeyBle() 함수를 호출하여 BLE 통신을 통해 차량을 제어 가능하도록 한다.
|objectDetailScreen.js|OnpressHorn() (onPressHazardLights(), onPressDoorUnlock(), onPressDoorLock()) | controlType과 Loader를 가진 함수. controlSmartKey() 함수에 인자로 controlType을 보내준다.
|-|BLE 통신|-|
|objectDetailScreen.js|initialize()|BLE 통신을 위한 BLE 초기화 함수.
|objectDetailScreen.js|connectBLE()|스마트폰과 차량 단말의 BLE를 연결시키는 함수.
|objectDetailScreen.js|controlSmartKeyBle()|controlType에 따라 actionBLE() 함수를 호출하여 스마트키를 조작 가능하게 하는 함수.
|objectDetailScreen.js|actionBLE()|controlType과 Loading을 전달하는 함수.
|-|-|-|
|SmartKeyView.js|renderActionButton()|context로 부터 받은 데이터들을 토대로 스마트키의 버튼들을 그리는 함수.

## ObjectDetailScreen.js
> - 차량의 상세 정보( 트립 정보, 예약정보 등)가 노출됩니다.
>- 해당 차량에 일반 고객의 예약이 없으면 fa가 차량을 제어할 수 있도록 스마트키가 활성화 됩니다.
> - 스마트키는 `서버와 통신`하여 차량 내에 삽입된 단말기의 제어를 통해 차량을 조작하며, 서버와의 통신 불가 시 차량 내 단말기와 사용자의 스마트폰 간의 `BLE 통신`을 통하여 차량을 제어할 수 있습니다.
> - 스마트키 조작을 위한 데이터들을 `React Context API`를 사용하여 관리합니다.

#### Code 

 - 스마트키 동작을 위한 API / BLE 데이터를 관리한 Context API 생성

	    const SmartKeyContext = createContext();

- 사용자가 스마트키의 차량 제어버튼 press 시 해당 동작이 수행되는 동안 노출될 Loader 초기화

	    const [isLoadingHorn, setLoadingHorn] = useState(false);
	    const [isLoadingHazardLight, setLoadingHazardLight] = useState(false);
	    const [isLoadingDoorUnlock, setLoadingDoorUnlock] = useState(false);
	    const [isLoadingDoorLock, setLoadingDoorLock] = useState(false);
- 스마트키 제어 API 호출

		// controlSmartKey function의 파라미터로 controlType을 지정
	    const  controlSmartKey = async (controlType) => {
	    
		// raidea-api의 /device-service/v1.1/control/object
	    const  response = await  SmartKeyControl.control(
	    tripData.terminalId, // 터미널 아이디
	    bookingId.current, // 예약 아이디 (null 또는 undefind로 설정하여도 차량 제어 가능)
	    controlType, // 차량 컨트롤 타입
	    'otoplug'  // 단말 제공자 ('otoplug'의 경우 차량의 가장 기본적은 제어만(도어 개패, 비상등, 경적) 가능)
	    );
	    이하 생략 ...
- control type 및 Loader
		
	    // control Type 경적
	    const  onPressHorn = async () => {
		    // 경적 버튼을 누름과 동시 useState hook을 사용 Loader를 활성화
		    setLoadingHorn(true); 
		    // controlSmartKey()의 인자로 경적울림의 controlType을 넘겨줌
		    controlSmartKey(SmartKeyControlType.horn);
		    // 해당 response를 받으면 Loader 비활성화
		    setLoadingHorn(false);
	    };
	    
	    // control type 비상등	         
	    const  onPressHazardLights = async () => {	    	    
		    setLoadingHazardLight(true);	    
		    controlSmartKey(SmartKeyControlType.light);	    
		    setLoadingHazardLight(false);	    
	    };
	    
	    // control type 문 잠금 해제	      	    
	    const  onPressDoorUnlock = async () => {	   	    
		    setLoadingDoorUnlock(true);	    
		    controlSmartKey(SmartKeyControlType.open);	    
		    setLoadingDoorUnlock(false);	    
	    };
	    	 
	    // control type 문 잠금   	    
	    const  onPressDoorLock = async () => {	    
		    setLoadingDoorLock(true);	    
		    controlSmartKey(SmartKeyControlType.close);	    
		    setLoadingDoorLock(false);	    
	    };
- context Provider의 value값 설정

	    const  value = {	    
		    onPressHorn,	    
		    onPressHazardLights,	    
		    onPressDoorUnlock,	    
		    onPressDoorLock,	    
		    isLoadingHorn,	    
		    isLoadingHazardLight,	    
		    isLoadingDoorUnlock,	    
		    isLoadingDoorLock,	    
	    };
- SmartKeyView를 Provider로 감싼 후 내보내기

	    <SmartKeyContext.Provider  value={value}>
		    {tripData && <SmartKeyView  />}	    
	    </SmartKeyContext.Provider>

	    export  default  ObjectDetailScreen;

## SmartKeyView.js
>- ObjectDetailView 하단에 스마트키의 UI를 그려줄 단순한 View 역할입니다.
>- Context로 관리되는 스마트키의 구동 데이터를 React Hook의 useContext를 사용해 해당 데이터로 차량을 제어하도록 합니다.

- import
	
	    // react hook api의 useContext를 사용하기 위한 import
	    import  React, {useContext} from  'react';
	    
	    // ObjectDetailScreen에서 생성한 Context를 사용하기 위한 import
	    import {SmartKeyContext} from  '../../screens/ObjectDetailScreen';
	    
	    // 화면에 그려질 스마트키의 아이콘
		import  iconHorn  from  '../../../assets/images/iconHorn.png';
		import  iconLight  from  '../../../assets/images/iconLight.png';
		import  iconOpen  from  '../../../assets/images/iconOpen.png';
		import  iconClose  from  '../../../assets/images/iconClose.png';
- useContext를 사용하여 데이터 가져오기
	
		// SmartKeyContext에서 value로 넘겨준 값을 받아옴
	    const  context = useContext(SmartKeyContext);
- 스마트키 UI

	    const  renderActionButton = (icon, action, isLoading) => {
		    return (	    
			    <TouchableOpacity	    
					style={{	    
					    flex: 1,	    
					    height: 60,	    
					    justifyContent: 'center',	    
					    alignItems: 'center',	    
					}}	    
					onPress={action}	    
				>	    
				    <Image	    
					    style={{	    
						    position: 'absolute',	    
						    opacity: isLoading ? 0.2 : 1,	    
					    }}	    
					    source={icon}	    
				    />	    
				    <ActivityIndicator	    
					    style={{	    
						    position: 'absolute',	    
					    }}	    
						    color={colorGuide.white}	    
						    animating={isLoading}	    
					    />	    
				    </TouchableOpacity>	    
			    );	    
		  };
- renderActionButton()에 icon, action, loading 넣기

	    return (	    
		    <View	    
			    style={{	    
				    backgroundColor:  colorGuide.azul,	    
				    flexDirection: 'row',	    
			    }}	    
		    >
			    {renderActionButton(	    
				    iconHorn,	    
				    context.onPressHorn,	    
				    context.isLoadingHorn	    
			    )}
			    
			    {renderActionButton(	    
				    iconLight,	    
				    context.onPressHazardLights,	    
				    context.isLoadingHazardLight	    
			    )}
			    
			    {renderActionButton(	    
				    iconOpen,	    
				    context.onPressDoorUnlock,	    
				    context.isLoadingDoorUnlock	    
			    )}
			    
			    {renderActionButton(	    
				    iconClose,	    
				    context.onPressDoorLock,	    
				    context.isLoadingDoorLock	    
			    )}
	    
		    </View>	    
	    );

 
