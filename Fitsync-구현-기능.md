# 요구사항

- 백엔드 구현 코드를 보고 포트폴리오 양식에 맞춰 작성
- 구현한 기능을 "문제 → 해결 → 결과" 를 중점으로 작성


# 현재 포트폴리오에 작성한 양식

## FitSync : 운동기록 관리 및 트레이너 매칭 서비스


**일정** : 2025.07.07 ~ 2025.08.13 (총 6주)

**기술 스택** : Spring, MyBatis, React, Java11, Javascript, Oracle

**GitHub** : [https://github.com/yourjinKR/myFitSync](https://github.com/yourjinKR/myFitSync)

%% 여기서부터 구현 내용을 작성한다. %%


# 구현 코드

## AI

### ApiLogServiceImple

```java
package org.fitsync.service;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

import org.fitsync.domain.AiExerciseDTO;
import org.fitsync.domain.AiRoutineDTO;
import org.fitsync.domain.ApiLogSearchCriteria;
import org.fitsync.domain.ApiLogStatsDTO;
import org.fitsync.domain.ApiLogVO;
import org.fitsync.domain.PtVO;
import org.fitsync.mapper.ApiLogMapper;
import org.fitsync.mapper.PaymentOrderMapper;
import org.fitsync.mapper.PtMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;

@Service
public class ApiLogServiceImple implements ApiLogService{
	@Autowired
	ApiLogMapper apiLogMapper;
	@Autowired
	PtMapper ptMapper;
	@Autowired
	PaymentOrderMapper payOrderMapper;
	
	@Override
	public void insertApiLog(ApiLogVO log) {
		apiLogMapper.insertApiLog(log);
		
	}
	
	@Override
	public ApiLogVO selectApiLogById(int apilog_idx) {
		return apiLogMapper.selectApiLogById(apilog_idx);
	}
	
	// idx와 이름 매핑하여 부르기 (허용된 운동)
	public Map<Integer, String> getWorkoutNameMap() {
	    List<PtVO> ptList = ptMapper.getWorkOutNameMap();
	    return ptList.stream().collect(Collectors.toMap(PtVO::getPt_idx, PtVO::getPt_name));
	}
	
	// idx와 이름 매핑하여 부르기 (모든 운동)
	public Map<Integer, String> getAllWorkoutNameMap() {
		List<PtVO> ptList = ptMapper.getAllWorkOutNameMap();
		return ptList.stream().collect(Collectors.toMap(PtVO::getPt_idx, PtVO::getPt_name));
	}
	
	@Override
	public List<ApiLogVO> selectApiList() {
	    List<ApiLogVO> list = apiLogMapper.selectApiList();

	    // 1. idx → name 매핑 Map 준비
	    Map<Integer, String> ptNameMap = getAllWorkoutNameMap();

	    // 2. Jackson ObjectMapper
	    ObjectMapper objectMapper = new ObjectMapper();

	    for (ApiLogVO log : list) {
	        String version = log.getApilog_version();  // ex: "0.2.0"
	        String json = log.getApilog_response();

	        // 1. 응답이 JSON이 아닌 경우 → 스킵
	        if (json == null || !json.trim().startsWith("[")) {
	            continue;
	        }

	        // 2. 버전별 파싱 분기
	        try {
	            if (version != null && version.startsWith("0.2")) {
	                // pt_idx만 존재 → 매핑 필요
	                List<AiRoutineDTO> routines = objectMapper.readValue(json, new TypeReference<List<AiRoutineDTO>>() {});
	                for (AiRoutineDTO routine : routines) {
	                    for (AiExerciseDTO ex : routine.getExercises()) {
	                        String name = ptNameMap.get(ex.getPt_idx());
	                        ex.setPt_name(name != null ? name : "Unknown");
	                    }
	                }
	                log.setApilog_response(objectMapper.writeValueAsString(routines));
	            }
	            // 0.1.x는 pt_name 포함된 구조이므로 그대로 유지
	        } catch (Exception e) {
	            log.setApilog_status("exception");
	            log.setApilog_status_reason("response_parsing_failed");
	            log.setApilog_response(json); // 원본 유지
	        }
	    }

	    return list;
	}
	
	@Override
	public int countTotalSubscriber() {
		return payOrderMapper.countActiveSubscribers();
	}
	
	@Override
	public ApiLogStatsDTO selectApiLogStats(ApiLogSearchCriteria cri) {
		return apiLogMapper.selectApiLogStats(cri);
	}
	
	@Override
	public void updateExceptionReason(ApiLogVO log) {
		apiLogMapper.updateExceptionReason(log);
	}
	
	@Override
	public void updateFeedBack(ApiLogVO apiLogVO) {
		apiLogMapper.updateFeedBack(apiLogVO);
	}
	
	@Override
	public int updateUserAction(ApiLogVO apiLogVO) {
		return apiLogMapper.updateUserAction(apiLogVO);
	}
	
	@Override
	public List<ApiLogVO> getByMemberId(int memberIdx) {
	    List<ApiLogVO> list = apiLogMapper.selectByMemberId(memberIdx);

	    // 1. idx → name 매핑 Map 준비
	    Map<Integer, String> ptNameMap = getAllWorkoutNameMap();

	    // 2. Jackson ObjectMapper
	    ObjectMapper objectMapper = new ObjectMapper();

	    for (ApiLogVO log : list) {
	        String version = log.getApilog_version();  // ex: "0.2.0"
	        String json = log.getApilog_response();

	        // 1. 응답이 JSON이 아닌 경우 → 스킵
	        if (json == null || !json.trim().startsWith("[")) {
	            continue;
	        }

	        // 2. 버전별 파싱 분기
	        try {
	            if (version != null && version.startsWith("0.2")) {
	                // pt_idx만 존재 → 매핑 필요
	                List<AiRoutineDTO> routines = objectMapper.readValue(json, new TypeReference<List<AiRoutineDTO>>() {});
	                for (AiRoutineDTO routine : routines) {
	                    for (AiExerciseDTO ex : routine.getExercises()) {
	                        String name = ptNameMap.get(ex.getPt_idx());
	                        ex.setPt_name(name != null ? name : "Unknown");
	                    }
	                }
	                log.setApilog_response(objectMapper.writeValueAsString(routines));
	            }
	            // 0.1.x는 pt_name 포함된 구조이므로 그대로 유지
	        } catch (Exception e) {
	            log.setApilog_status("exception");
	            log.setApilog_status_reason("response_parsing_failed");
	            log.setApilog_response(json); // 원본 유지
	        }
	    }

	    return list;
	}
}

```

### AIServiceImple

```java
package org.fitsync.service;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.math.BigDecimal;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.sql.Timestamp;
import java.time.LocalDateTime;
import java.util.*;
import java.util.stream.Collectors;

import org.fitsync.domain.AiExerciseDTO;
import org.fitsync.domain.AiRoutineDTO;
import org.fitsync.domain.ApiLogVO;
import org.fitsync.domain.ApiResponseDTO;
import org.fitsync.domain.PtVO;
import org.fitsync.mapper.ApiLogMapper;
import org.fitsync.mapper.PtMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.GetMapping;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

@Service
public class AIServiceImple implements AIService {

    @Value("${chatgpt.api.key}")
    private String apiKey;

    private static final String API_URL = "https://api.openai.com/v1/chat/completions";

    @Autowired
    private ApiLogMapper apiLogMapper;
	@Autowired
	private PtMapper ptMapper;
	
	public String getWorkoutNamesJsonArray() {
	    List<String> names = ptMapper.getWorkOutName();
	    String jsonArray = names.stream()
	        .map(name -> "\"" + name + "\"")
	        .collect(Collectors.joining(", ", "[", "]"));
	    return jsonArray;
	}
	
	public String getWorkoutNamesCommaSeparated() {
	    List<String> names = ptMapper.getWorkOutName();
	    return names.stream()
	                .collect(Collectors.joining(", "));
	}
	
	// idx와 이름 매핑하여 부르기
	public Map<Integer, String> getWorkoutNameMap() {
	    List<PtVO> ptList = ptMapper.getWorkOutNameMap();
	    return ptList.stream().collect(Collectors.toMap(PtVO::getPt_idx, PtVO::getPt_name));
	}
	
	// JSON으로 변환
	public String getWorkoutMapForPrompt() {
	    Map<Integer, String> map = getWorkoutNameMap();

	    StringBuilder sb = new StringBuilder("운동 목록:\n[");
	    map.forEach((idx, name) -> {
	        sb.append(String.format("{pt_idx: %d, pt_name: \"%s\"}, ", idx, name));
	    });

	    if (!map.isEmpty()) sb.setLength(sb.length() - 2); // 마지막 쉼표 제거
	    sb.append("]");
	    return sb.toString();
	}
	

    @Override
    public ApiResponseDTO requestAIResponse(String userMessage, int memberIdx) throws IOException {    	
    	ObjectMapper objectMapper = new ObjectMapper();
    	
    	Map<String, Object> map = objectMapper.readValue(userMessage, new TypeReference<Map<String, Object>>() {});
    	int userSplit = (int) map.get("split");
    	
    	
        Timestamp requestTime = new Timestamp(System.currentTimeMillis());
        String workoutListJson = getWorkoutMapForPrompt();
        Integer logIdx = null;
        String finalResponseJson = null;
        ApiLogVO apiLog = new ApiLogVO();

        String content = "";
        String status = "success";
        List<String> exceptionReasons = new ArrayList<>();
        String errorMessage = null;
        int inputTokens = 0;
        int outputTokens = 0;
        Timestamp responseTime = null;
        String apiModel = "gpt-4o";

        // 1. 메시지 구성
        String systemContent =
		    "너는 퍼스널 트레이너야. 아래 사용자 정보(JSON)를 기반으로, 분할 루틴을 추천해.\n\n" +
		    "사용자 정보는 다음 필드를 포함해:\n" +
		    "- age: 사용자 나이 (정수)\n" +
		    "- gender : 성별 (남성, 여성)" +
		    "- height: 키 (cm)\n" +
		    "- weight: 몸무게 (kg)\n" +
		    "- bmi: 체질량지수\n" +
		    "- fat: 체지방량 (kg)\n" +
		    "- fat_percentage: 체지방률 (%)\n" +
		    "- skeletal_muscle: 골격근량 (kg)\n" +
		    "- disease: 사용자가 불편한 신체 부위 (예: [무릎, 발목...])\n" +
		    "- purpose: 운동 목적 (예: 다이어트, 근력 증가, 체형 교정 등)\n" +
		    "- day: 운동 가능한 요일 (예: 월, 수, 금)\n" +
		    "- time: 운동 가능한 시간대 (예: 오전, 오후, 저녁)\n" +
		    "- split: 사용자가 원하는 루틴 분할 수 (예: 3이면 3분할 루틴 생성)\n\n" +
		    "이 정보들을 기반으로 루틴을 작성하고, 응답은 반드시 JSON 형식으로만 작성하고, 어떤 설명이나 텍스트도 포함 금지. 마크다운 또한 금지\n" +
		    "루틴은 분할 수에 맞춰 나눠야 하며, 각 루틴은 운동 4~6개, 1시간 분량으로 구성해.\n" +
		    "운동 목록은 다음과 같아. 반드시 아래 pt_idx 중에서만 선택해서 추천해. 응답 시 pt_name 대신 pt_idx로만 응답해야 해:\n" +
		    "운동 목록  :" + workoutListJson +"\n" +
		    "각 운동은 아래 항목을 포함해야 해:\n" + 
		    "- pt_idx: 운동 ID (정수)\n" +
		    "- set_volume: 중량 또는 시간 (중량이 필요한 운동은 숫자만 입력하고 단위 없이 kg 기준, 유산소 운동과 같이 시간이 필요한 경우 초 단위로 입력하되 단위 생략. 반드시 숫자로만 출력.)\n" +
		    "- set_count: 횟수\n" +
		    "- set_num: 세트 수\n\n" +
		    "형식 예시:\n" +
		    "[\n" +
		    "  {\n" +
		    "    \"routine_name\": \"가슴 등 루틴\",\n" +
		    "    \"exercises\": [\n" +
		    "      {\"pt_idx\": 131, \"set_volume\": 60, \"set_count\": 10, \"set_num\": 4},\n" +
		    "      {\"pt_idx\": 215, \"set_volume\": 50, \"set_count\": 10, \"set_num\": 4}\n" +
		    "    ]\n" +
		    "  },\n" +
		    "  {\n" +
		    "    \"routine_name\": \"하체 루틴\",\n" +
		    "    \"exercises\": [\n" +
		    "      {\"pt_idx\": 3, \"set_volume\": 80, \"set_count\": 10, \"set_num\": 4},\n" +
		    "      {\"pt_idx\": 21, \"set_volume\": 100, \"set_count\": 10, \"set_num\": 4}\n" +
		    "    ]\n" +
		    "  }\n" +
		    "]";

        Map<String, Object> systemMessage = new HashMap<>();
        systemMessage.put("role", "system");
        systemMessage.put("content", systemContent);

        Map<String, Object> userMessageMap = new HashMap<>();
        userMessageMap.put("role", "user");
        userMessageMap.put("content", userMessage);

        List<Map<String, Object>> messages = new ArrayList<>();
        messages.add(systemMessage);
        messages.add(userMessageMap);

        Map<String, Object> body = new HashMap<>();
        body.put("model", apiModel);
        body.put("messages", messages);

        String requestBody = objectMapper.writeValueAsString(body);

        try {
            URL url = new URL(API_URL);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("POST");
            connection.setRequestProperty("Authorization", "Bearer " + apiKey);
            connection.setRequestProperty("Content-Type", "application/json");
            connection.setDoOutput(true);

            try (OutputStream os = connection.getOutputStream()) {
                os.write(requestBody.getBytes(StandardCharsets.UTF_8));
            }

            StringBuilder responseBuilder = new StringBuilder();
            try (BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
                String line;
                while ((line = br.readLine()) != null) {
                    responseBuilder.append(line);
                }
            }

            responseTime = new Timestamp(System.currentTimeMillis());

            JsonNode root = objectMapper.readTree(responseBuilder.toString());
            content = root.path("choices").get(0).path("message").path("content").asText();
            inputTokens = root.path("usage").path("prompt_tokens").asInt();
            outputTokens = root.path("usage").path("completion_tokens").asInt();

            // 1. AI 응답 JSON 파싱
            List<AiRoutineDTO> aiRoutines = objectMapper.readValue(content, new TypeReference<List<AiRoutineDTO>>() {});

            // 2. PT 이름 맵핑 정보 로드 (DB 1회 호출)
            Map<Integer, String> ptNameMap = getWorkoutNameMap();
            
            
            List<Integer> unknownPtIdxList = new ArrayList<>();
            // 3. pt_idx → pt_name 매핑 수행
            for (AiRoutineDTO routine : aiRoutines) {
                for (AiExerciseDTO exercise : routine.getExercises()) {
                    String ptName = ptNameMap.get(exercise.getPt_idx());
                    if (ptName != null) {
                        exercise.setPt_name(ptName);
                    } else {
                        System.err.println("⚠️ Unknown pt_idx: " + exercise.getPt_idx());
                        exercise.setPt_name("Unknown");
                        unknownPtIdxList.add(exercise.getPt_idx());
                    }
                }
            }
            
            // 잘못된 idx가 응답했을 경우 (invalid_exercise)
            if (!unknownPtIdxList.isEmpty()) {
                exceptionReasons.add("invalid_exercise: unknown pt_idx(s) = " + unknownPtIdxList);
            }
            // 사용자의 분할 수 요청과 응답이 다를 경우 (invalid_exercise)
            if (userSplit != aiRoutines.size()) {
            	exceptionReasons.add("split_mismatch: expected=" + userSplit + ", actual=" + aiRoutines.size());
            }
            // 예외 최종 기록
            if (!exceptionReasons.isEmpty()) {
                apiLog.setApilog_status("exception");
                apiLog.setApilog_status_reason(String.join("; ", exceptionReasons));
            }
 
            // 4. 매핑이 완료된 aiRoutines → JSON 문자열로 다시 변환
            finalResponseJson = objectMapper.writeValueAsString(aiRoutines);
            

        } catch (IOException e) {
            responseTime = new Timestamp(System.currentTimeMillis());
            status = "fail";
            content = e.getMessage();
        }

        // 로그 저장
        try {
            apiLog.setMember_idx(memberIdx);
            apiLog.setApilog_prompt(requestBody);
            apiLog.setApilog_response(content);
            apiLog.setApilog_request_time(requestTime);
            apiLog.setApilog_response_time(responseTime);
            apiLog.setApilog_input_tokens(inputTokens);
            apiLog.setApilog_output_tokens(outputTokens);
            apiLog.setApilog_model(apiModel);
            apiLog.setApilog_version("0.2.1");
            apiLog.setApilog_status(status);
            apiLog.setApilog_service_type("사용자 정보 기반 운동 루틴 추천");

            apiLogMapper.insertApiLog(apiLog);
            
            logIdx = apiLog.getApilog_idx();
            
        } catch (Exception logEx) {
            System.err.println("로그 저장 실패: " + logEx.getMessage());
        }

        if ("fail".equals(status)) {
            throw new IOException("GPT 요청 실패: " + finalResponseJson);
        }

        return new ApiResponseDTO(finalResponseJson, logIdx);
    }

    // 운동기록 분석기반 AI 피드백 서비스
    @Override
    public ApiResponseDTO requestAIfeedback(String userMessage, int memberIdx) throws Exception {
        /**
         * 기존에 루틴 생성 AI 서비스만으로는 부족함을 느낌.
         * 사용자가 지속적인 케어를 받을 수 있는 AI 피드백 서비스를 기획 중임.
         * AI가 사용자의 정보를 받아 분석하고 운동에 방향성을 피드백해줌.
         * 일일 단위로 지원되는 AI 서비스로 예상됨.
         * 
         * 사용자로부터 받는 데이터
         * 1. 오늘 한 운동 (recordVO)
         * 2. 지난 운동들 (List<recordVO>)
         * 3. 사용자 정보 (MemberVO, BodyVO)
         * 4. 지난 사용자 정보 (필요할지 의문임)
         */


        return null;
    }

}

```

## 결제

### ScheduledPaymentMonitor

```java
package org.fitsync.service;

import org.fitsync.domain.PaymentOrderVO;
import org.fitsync.domain.PaymentOrderWithMethodVO;
import org.fitsync.mapper.PaymentOrderMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import com.fasterxml.jackson.databind.ObjectMapper;

import lombok.extern.log4j.Log4j;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.sql.Timestamp;
import java.time.Duration;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;

@Component
@Log4j
public class ScheduledPaymentMonitor {
    
    @Autowired
    private PaymentOrderMapper paymentOrderMapper;
    
    @Autowired
    private PaymentService paymentService;
    
    @Value("${portone.api.secret}")
    private String apiSecretKey;
    
    @Value("${portone.store.id}")
    private String storeId;
    
    /**
     * 모니터링 활성화 여부 (기본값: false)
     * 마스터 서버에서만 true로 설정
     */
    @Value("${payment.monitor.enabled:false}")
    private boolean monitorEnabled;
    
    /**
     * 자동 예약 기능 활성화 여부 (기본값: true)
     */
    @Value("${payment.auto.schedule.enabled:true}")
    private boolean autoScheduleEnabled;
    
    /**
     * 서버 식별용 이름 (로깅용)
     */
    @Value("${server.name:unknown-server}")
    private String serverName;
    
    /**
     * API 호출 제한 (분당 최대 호출 수)
     */
    @Value("${payment.monitor.api.max.calls.per.minute:15}")
    private int maxApiCallsPerMinute;
    
    /**
     * API 호출 간격 (밀리초)
     */
    @Value("${payment.monitor.api.delay.ms:1000}")
    private long apiDelayMs;
    
    /**
     * 모니터링 시간 범위 (분)
     */
    @Value("${payment.monitor.time.range.minutes:10}")
    private int timeRangeMinutes;
    
    // ObjectMapper 재사용을 위한 필드 (메모리 효율성)
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    // HttpClient 재사용 (타임아웃 설정 포함)
    private final HttpClient httpClient = HttpClient.newBuilder()
        .connectTimeout(Duration.ofSeconds(10))  // 연결 타임아웃 10초
        .build();
    
    // API 호출 횟수 카운터 (분당 제한용)
    private final AtomicInteger currentApiCallCount = new AtomicInteger(0);
    
    /**
     * 일일 배치 - 예약 결제 상태 모니터링 (매일 00시 10분에 실행)
     * 당일(00:00:00 ~ 23:59:59) 예약된 모든 결제를 일괄 처리
     * 마스터 서버에서만 실행됨
     */
    @Scheduled(cron = "0 10 0 * * ?")
    public void processDailyPaymentBatch() {
        
        // 모니터링이 비활성화된 서버는 실행하지 않음
        if (!monitorEnabled) {
            log.debug("일일 배치 비활성화 서버 (" + serverName + ") - 스케줄러 건너뛰기");
            return;
        }
        
        // API 호출 카운터 초기화
        currentApiCallCount.set(0);
        
        long startTime = System.currentTimeMillis();
        java.time.LocalDate today = java.time.LocalDate.now(java.time.ZoneId.of("Asia/Seoul"));
        
        try {
            log.info("=== 일일 결제 배치 처리 시작 (날짜: " + today + ", 서버: " + serverName + ") ===");
            System.out.println("[" + serverName + "] " + today + " 일일 결제 배치 시작");
            
            // 1. 당일(00:00:00 ~ 23:59:59) 예약 결제 조회
            java.time.LocalDateTime todayStart = today.atTime(0, 0, 0);
            java.time.LocalDateTime todayEnd = today.atTime(23, 59, 59);
            
            Timestamp batchStart = Timestamp.valueOf(todayStart);
            Timestamp batchEnd = Timestamp.valueOf(todayEnd);
            
            log.info("배치 처리 범위: " + todayStart + " ~ " + todayEnd);
            
            // 2. 당일 예약 결제 조회 (정각에 설정된 예약들)
            List<PaymentOrderVO> todayScheduledOrders = paymentOrderMapper
                .selectScheduledPaymentsByTimeRange(batchStart, batchEnd);
            
            if (todayScheduledOrders.isEmpty()) {
                log.info("당일 처리할 예약 결제가 없습니다. (날짜: " + today + ")");
                System.out.println("[" + serverName + "] 당일 처리할 예약 결제 없음");
                return;
            }
            
            log.info("📋 당일 처리 대상 예약 결제: " + todayScheduledOrders.size() + "건");
            System.out.println("📋 [" + serverName + "] 당일 처리 대상: " + todayScheduledOrders.size() + "건");
            
            // 3. 각 예약 결제를 일괄 처리
            int totalProcessed = 0;
            int successCount = 0;
            int failureCount = 0;
            int unchangedCount = 0;
            int skippedCount = 0;
            
            log.info("🔄 일괄 처리 시작...");
            
            for (PaymentOrderVO order : todayScheduledOrders) {
                totalProcessed++;
                
                // API 호출 제한 체크 (안전장치)
                if (currentApiCallCount.get() >= maxApiCallsPerMinute * 2) { // 배치용으로 제한 완화
                    log.warn("⚠️ API 호출 제한 초과 - 남은 " + (todayScheduledOrders.size() - totalProcessed) + "건은 다음 배치에서 처리");
                    skippedCount = todayScheduledOrders.size() - totalProcessed + 1;
                    break;
                }
                
                String result = checkAndUpdateScheduledPayment(order);
                
                switch (result) {
                    case "SUCCESS": 
                        successCount++; 
                        System.out.println("[배치] 결제 성공 - OrderIdx: " + order.getOrder_idx());
                        break;
                    case "FAILED": 
                        failureCount++; 
                        System.out.println("[배치] 결제 실패 - OrderIdx: " + order.getOrder_idx());
                        break;
                    case "UNCHANGED": 
                        unchangedCount++; 
                        System.out.println("[배치] 대기 중 - OrderIdx: " + order.getOrder_idx());
                        break;
                    case "API_LIMIT_EXCEEDED": 
                        skippedCount++; 
                        break;
                }
                
                // 배치 처리 간격 조절 (API 부하 방지)
                if (totalProcessed < todayScheduledOrders.size()) {
                    try {
                        Thread.sleep(500); // 0.5초 간격 (배치용으로 단축)
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        log.warn("배치 처리 간격 대기 중 인터럽트 발생");
                        break;
                    }
                }
            }
            
            long executionTime = System.currentTimeMillis() - startTime;
            
            log.info("=== 일일 결제 배치 처리 완료 (날짜: " + today + ", 서버: " + serverName + 
                    ", 실행시간: " + executionTime + "ms) ===");
            log.info("처리 결과 - 총 처리: " + totalProcessed + "건 중 " + 
                    "성공: " + successCount + "건, 실패: " + failureCount + "건, " + 
                    "대기: " + unchangedCount + "건, 건너뜀: " + skippedCount + "건");
            
            System.out.println("[" + serverName + "] " + today + " 일일 배치 완료!");
            System.out.println("[결과] 성공: " + successCount + "건, 실패: " + failureCount + "건, " + 
                             "대기: " + unchangedCount + "건 (총 " + totalProcessed + "건 처리)");
            
            // 성과 요약 로깅
            if (successCount > 0 || failureCount > 0) {
                System.out.println("[" + serverName + "] " + today + " 결제 처리: " + 
                                 successCount + "건 완료, " + failureCount + "건 실패");
            }
            
        } catch (Exception e) {
            long executionTime = System.currentTimeMillis() - startTime;
            log.error("일일 결제 배치 처리 중 오류 발생 (날짜: " + today + ", 서버: " + serverName + 
                     ", 실행시간: " + executionTime + "ms): ", e);
            System.err.println("[" + serverName + "] 일일 배치 오류: " + e.getMessage());
        }
    }
    
    /**
     * 개별 예약 결제 상태 확인 및 업데이트
     * API 호출과 DB 업데이트를 분리하여 트랜잭션 최적화
     */
    public String checkAndUpdateScheduledPayment(PaymentOrderVO order) {
        
        // API 호출 제한 체크
        if (currentApiCallCount.get() >= maxApiCallsPerMinute) {
            log.warn("API 호출 제한 초과 - OrderIdx: " + order.getOrder_idx());
            return "API_LIMIT_EXCEEDED";
        }
        
        try {
            String scheduleId = order.getSchedule_id();
            
            log.info("예약 결제 상태 확인 - OrderIdx: " + order.getOrder_idx() + 
                ", ScheduleId: " + scheduleId + ", ScheduleDate: " + order.getSchedule_date());
            
            // 1. PortOne API로 예약 상태 조회 (트랜잭션 외부에서 실행)
            String apiResponseBody = callPortOneScheduleAPI(scheduleId);
            
            if (apiResponseBody == null) {
                return "FAILED"; // API 호출 실패
            }
            
            // 2. API 응답 파싱
            @SuppressWarnings("unchecked")
            Map<String, Object> responseData = objectMapper.readValue(apiResponseBody, Map.class);
            String apiStatus = (String) responseData.get("status");
            
            log.debug("API 응답 상태 - ScheduleId: " + scheduleId + ", Status: " + apiStatus);
            
            // 3. 상태 변경이 필요한 경우에만 DB 트랜잭션 실행
            return processStatusUpdate(order, apiStatus, responseData);
            
        } catch (Exception e) {
            log.error("예약 결제 상태 확인 중 오류 - OrderIdx: " + order.getOrder_idx(), e);
            return "FAILED";
        }
    }
    
    /**
     * PortOne API 호출 (타임아웃 및 재사용 최적화)
     */
    private String callPortOneScheduleAPI(String scheduleId) {
        try {
            // API 호출 카운터 증가
            currentApiCallCount.incrementAndGet();
            
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.portone.io/payment-schedules/" + scheduleId + "?storeId=" + storeId))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + apiSecretKey)
                .timeout(Duration.ofSeconds(15)) // 응답 타임아웃 15초
                .method("GET", HttpRequest.BodyPublishers.noBody())
                .build();
            
            HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            
            if (response.statusCode() >= 200 && response.statusCode() < 300) {
                return response.body();
            } else {
                log.warn("예약 상태 조회 실패 - ScheduleId: " + scheduleId + 
                    ", StatusCode: " + response.statusCode() + ", Body: " + response.body());
                return null;
            }
            
        } catch (java.net.http.HttpTimeoutException e) {
            log.error("API 호출 타임아웃 - ScheduleId: " + scheduleId, e);
            return null;
        } catch (java.io.IOException e) {
            log.error("API 호출 네트워크 오류 - ScheduleId: " + scheduleId, e);
            return null;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            log.error("API 호출 인터럽트 - ScheduleId: " + scheduleId, e);
            return null;
        } catch (Exception e) {
            log.error("API 호출 중 예상치 못한 오류 - ScheduleId: " + scheduleId, e);
            return null;
        }
    }
    
    /**
     * 상태 업데이트 처리 (필요한 경우에만 짧은 트랜잭션 실행)
     */
    private String processStatusUpdate(PaymentOrderVO order, String apiStatus, Map<String, Object> responseData) {
        
        String newStatus = null;
        String resultType = "UNCHANGED";
        
        // 상태 매핑
        switch (apiStatus != null ? apiStatus : "") {
            case "SUCCEEDED":
                newStatus = "PAID";
                resultType = "SUCCESS";
                break;
            case "FAILED":
                newStatus = "FAILED";
                resultType = "FAILED";
                break;
            case "REVOKED":
                newStatus = "CANCELLED";
                resultType = "FAILED";
                break;
            case "SCHEDULED":
            case "PENDING":
                // 아직 대기 상태 - 상태 변경 없음
                log.debug("⏳ 예약 결제 대기 중 - OrderIdx: " + order.getOrder_idx() + ", API Status: " + apiStatus);
                return "UNCHANGED";
            default:
                log.warn("❓ 알 수 없는 API 상태 - OrderIdx: " + order.getOrder_idx() + ", Status: " + apiStatus);
                return "UNCHANGED";
        }
        
        // 상태 변경이 필요한 경우에만 트랜잭션 실행
        if (newStatus != null) {
            try {
                updateOrderStatusInTransaction(order, newStatus, responseData);
                
                // 로깅
                switch (newStatus) {
                    case "PAID":
                        log.info("예약 결제 성공 감지 - OrderIdx: " + order.getOrder_idx() + ", 상태 변경: READY -> PAID");
                        System.out.println("[" + serverName + "] 결제 성공! OrderIdx: " + order.getOrder_idx());
                        break;
                    case "FAILED":
                        log.info("예약 결제 실패 감지 - OrderIdx: " + order.getOrder_idx() + ", 상태 변경: READY -> FAILED");
                        System.out.println("[" + serverName + "] 결제 실패! OrderIdx: " + order.getOrder_idx());
                        break;
                    case "CANCELLED":
                        log.info("예약 결제 취소 감지 - OrderIdx: " + order.getOrder_idx() + ", 상태 변경: READY -> CANCELLED");
                        System.out.println("[" + serverName + "] 결제 취소! OrderIdx: " + order.getOrder_idx());
                        break;
                }
                
                // 알림 처리
                sendPaymentNotification(order, newStatus);
                
            } catch (Exception e) {
                log.error("상태 업데이트 실패 - OrderIdx: " + order.getOrder_idx(), e);
                return "FAILED";
            }
        }
        
        return resultType;
    }
    
    /**
     * DB 상태 업데이트 전용 트랜잭션 (짧고 효율적)
     */
    @Transactional(rollbackFor = Exception.class)
    public void updateOrderStatusInTransaction(PaymentOrderVO order, String newStatus, Map<String, Object> responseData) {
        
        // 업데이트 전 현재 상태 확인 (동시성 체크)
        PaymentOrderVO currentOrder = paymentOrderMapper.selectByOrderIdx(order.getOrder_idx());
        
        if (!"READY".equals(currentOrder.getOrder_status())) {
            log.warn("이미 처리된 주문 - OrderIdx: " + order.getOrder_idx() + 
                ", 현재 상태: " + currentOrder.getOrder_status() + ", 요청 상태: " + newStatus);
            return; // 이미 처리됨
        }
        
        // 주문 정보 업데이트
        order.setOrder_status(newStatus);
        
        if ("PAID".equals(newStatus)) {
            order.setOrder_paydate(new Timestamp(System.currentTimeMillis()));
            
            // 실제 결제 정보 저장
            @SuppressWarnings("unchecked")
            List<Map<String, Object>> payments = (List<Map<String, Object>>) responseData.get("payments");
            if (payments != null && !payments.isEmpty()) {
                Map<String, Object> payment = payments.get(0);
                String actualPaymentId = (String) payment.get("id");
                if (actualPaymentId != null) {
                    order.setPayment_id(actualPaymentId);
                }
            }
        }
        
        // 조건부 업데이트 실행 (현재 상태가 READY일 때만)
        int updatedRows = paymentOrderMapper.updatePaymentStatusConditional(order);
        
        if (updatedRows == 0) {
            log.warn("동시성 이슈로 업데이트 실패 - 다른 프로세스에서 이미 처리됨: OrderIdx " + order.getOrder_idx());
            throw new RuntimeException("Concurrent modification detected");
        }
        
        System.out.println("예약 결제 상태 업데이트 완료 - OrderIdx: " + order.getOrder_idx() + ", READY -> " + newStatus);
    }

    /**
     * 결제 완료/실패 알림 처리 및 정기 결제 자동 예약
     */
    private void sendPaymentNotification(PaymentOrderVO order, String status) {
        try {
            log.info("결제 알림 발송 - OrderIdx: " + order.getOrder_idx() + ", Status: " + status + ", MemberIdx: " + order.getMember_idx());
            
            // 결제 성공 시 추가 비즈니스 로직
            if ("PAID".equals(status)) {
                log.info("구독 활성화 처리 - MemberIdx: " + order.getMember_idx());
                
                // 정기 결제인 경우 다음 달 자동 예약 처리 (설정으로 제어)
                if ("SCHEDULE".equals(order.getOrder_type()) && autoScheduleEnabled) {
                    scheduleNextMonthAutoPayment(order);
                } else if ("SCHEDULE".equals(order.getOrder_type()) && !autoScheduleEnabled) {
                    log.info("자동 예약 기능이 비활성화됨 - MemberIdx: " + order.getMember_idx());
                }
                
                // TODO: 구독 활성화 로직 구현
            }
            
        } catch (Exception e) {
            log.error("알림 발송 중 오류 발생 - OrderIdx: " + order.getOrder_idx(), e);
            // 알림 실패는 전체 프로세스에 영향주지 않음
        }
    }
    
    /**
     * 다음 달 자동 결제 예약 처리 (정기 결제용)
     * 결제 성공 시 비동기로 다음 달 결제를 자동 예약
     */
    private void scheduleNextMonthAutoPayment(PaymentOrderVO completedOrder) {
        try {
            log.info("🔄 다음 달 자동 결제 예약 시작 - OrderIdx: " + completedOrder.getOrder_idx() + 
                    ", MemberIdx: " + completedOrder.getMember_idx());
            
            // 기존 다음 달 예약이 있는지 확인 (중복 예약 방지)
            PaymentOrderWithMethodVO existingSchedule = paymentOrderMapper.selectScheduledPaymentOrderByMember(completedOrder.getMember_idx());
            
            if (existingSchedule != null && !"CANCELLED".equals(existingSchedule.getOrder_status())) {
                log.info("이미 다음 달 예약이 존재함 - ExistingOrderIdx: " + existingSchedule.getOrder_idx() + 
                        ", Status: " + existingSchedule.getOrder_status());
                System.out.println("[자동 예약] 이미 다음 달 예약 존재 - MemberIdx: " + completedOrder.getMember_idx());
                return;
            }
            
            // PaymentService를 통해 다음 달 결제 예약
            Object scheduleResult = paymentService.scheduleNextMonthPayment(completedOrder);
            
            @SuppressWarnings("unchecked")
            Map<String, Object> result = (Map<String, Object>) scheduleResult;
            boolean isSuccess = (boolean) result.get("success");
            
            if (isSuccess) {
                log.info("다음 달 자동 결제 예약 성공 - OriginalOrderIdx: " + completedOrder.getOrder_idx() + 
                        ", NewScheduleId: " + result.get("scheduleId") + ", NextPaymentDate: " + result.get("nextPaymentDate"));
                System.out.println("[자동 예약] 성공! MemberIdx: " + completedOrder.getMember_idx() + 
                        ", 다음 결제일: " + result.get("nextPaymentDate"));
            } else {
                log.error("다음 달 자동 결제 예약 실패 - OriginalOrderIdx: " + completedOrder.getOrder_idx() + 
                        ", Error: " + result.get("message"));
                System.err.println("[자동 예약] 실패! MemberIdx: " + completedOrder.getMember_idx() + 
                        ", 이유: " + result.get("message"));
            }
            
        } catch (Exception e) {
            log.error("다음 달 자동 결제 예약 처리 중 오류 발생 - OrderIdx: " + completedOrder.getOrder_idx(), e);
            System.err.println("[자동 예약] 오류 발생 - MemberIdx: " + completedOrder.getMember_idx() + 
                    ", 오류: " + e.getMessage());
        }
    }
    
    /**
     * 모니터링 상태 확인용 메서드 (디버깅/관리용)
     */
    public String getMonitorStatus() {
        return "서버명: " + serverName + 
               ", 모니터링 활성화: " + monitorEnabled + 
               ", 자동 예약 활성화: " + autoScheduleEnabled + 
               ", 현재 API 호출 횟수: " + currentApiCallCount.get() + "/" + maxApiCallsPerMinute;
    }
}

```

### PaymentServiceImple

```java
package org.fitsync.service;

import java.net.http.HttpResponse;
import java.time.ZoneId;
import java.util.*;

import org.fitsync.domain.ApiLogVO;
import org.fitsync.domain.PaymentMethodVO;
import org.fitsync.domain.PaymentOrderVO;
import org.fitsync.domain.PaymentOrderWithMethodVO;
import org.fitsync.mapper.ApiLogMapper;
import org.fitsync.mapper.PaymentMethodMapper;
import org.fitsync.mapper.PaymentOrderMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.fasterxml.jackson.databind.ObjectMapper;

import lombok.extern.log4j.Log4j;

import java.io.IOException;
import java.math.BigDecimal;

@Log4j
@Service
public class PaymentServiceImple implements PaymentService {
    
	@Autowired
	private PaymentMethodMapper paymentMethodMapper;
	
	@Autowired
	private PaymentOrderMapper paymentOrderMapper;
	
	@Autowired
	private PortOneApiClient portOneApiClient;

	@Autowired
	private ApiLogMapper apiLogMapper;
	
	@Value("${payment.subscribe.cost}")
    private int subscribeCost;
	
	/**
	 * DB 연결 및 매퍼 상태 테스트
	 */
	public Map<String, Object> testDatabaseConnection() {
		Map<String, Object> result = new HashMap<>();
		
		try {
			log.info("=== DB 연결 테스트 시작 ===");
			
			// 매퍼 null 체크
			result.put("paymentMethodMapper", paymentMethodMapper != null ? "OK" : "NULL");
			result.put("paymentOrderMapper", paymentOrderMapper != null ? "OK" : "NULL");
			result.put("portOneApiClient", portOneApiClient != null ? "OK" : "NULL");
			
			// 간단한 조회 테스트 (존재하지 않는 memberIdx로 테스트)
			try {
				List<PaymentMethodVO> testMethods = paymentMethodMapper.selectByMemberIdx(99999);
				result.put("paymentMethodQuery", "SUCCESS - Count: " + (testMethods != null ? testMethods.size() : "NULL"));
			} catch (Exception e) {
				result.put("paymentMethodQuery", "FAILED - " + e.getMessage());
			}
			
			try {
				List<PaymentOrderVO> testOrders = paymentOrderMapper.selectPaymentOrdersByMember(99999);
				result.put("paymentOrderQuery", "SUCCESS - Count: " + (testOrders != null ? testOrders.size() : "NULL"));
			} catch (Exception e) {
				result.put("paymentOrderQuery", "FAILED - " + e.getMessage());
			}
			
			result.put("overallStatus", "COMPLETED");
			log.info("DB 연결 테스트 결과: " + result);
			
		} catch (Exception e) {
			log.error("DB 연결 테스트 중 오류: ", e);
			result.put("overallStatus", "ERROR");
			result.put("error", e.getMessage());
		}
		
		return result;
	}
	
	// 결제수단 등록 (카드 정보 포함)
	@Override
	public int saveBillingKey(PaymentMethodVO vo) {
		try {
			// 빌링키로 카드 정보 조회
			Map<String, Object> cardInfo = getCardInfoByBillingKey(vo.getMethod_key());
			
			// methodType 확인
			String methodType = (String) cardInfo.get("methodType");
			
			// 카드 결제인 경우에만 카드 정보 설정
			if ("card".equals(methodType)) {
				String cardName = (String) cardInfo.get("name");
				String cardNumber = (String) cardInfo.get("number");
				
				vo.setMethod_card(cardName != null ? cardName : "알 수 없는 카드");
				vo.setMethod_card_num(cardNumber != null ? cardNumber : "****-****-****-****");
				
				log.info("카드 정보와 함께 결제수단 저장: " + cardName + " (" + cardNumber + ")");
			} else {
				// 간편결제 등 카드가 아닌 경우 null로 설정
				vo.setMethod_card(null);
				vo.setMethod_card_num(null);
				
				log.info("간편결제 수단 저장 - 타입: " + methodType);
			}
			
			return paymentMethodMapper.insertPaymentMethod(vo);
			
		} catch (Exception e) {
			log.error("결제수단 저장 중 오류 발생: ", e);
			// 카드 정보 조회 실패 시에도 기본값으로 저장 시도
			vo.setMethod_card("정보 조회 실패");
			vo.setMethod_card_num("****-****-****-****");
			return paymentMethodMapper.insertPaymentMethod(vo);
		}
	}
	
	// 결제수단 불러오기 (빌링키 제외)
	@Override
	public List<PaymentMethodVO> getPaymentMethods(int memberIdx) {
		try {
			log.info("=== 결제수단 조회 시작 ===");
			log.info("Member ID: " + memberIdx);
			
			if (paymentMethodMapper == null) {
				log.error("PaymentMethodMapper가 null입니다!");
				throw new RuntimeException("PaymentMethodMapper가 초기화되지 않았습니다.");
			}
			
			List<PaymentMethodVO> methods = paymentMethodMapper.selectByMemberIdx(memberIdx);
			
			if (methods == null) {
				log.warn("결제수단이 null로 반환되었습니다.");
				return new ArrayList<>();
			}
			
			log.info("결제수단 조회 완료 - memberIdx: " + memberIdx + ", 건수: " + methods.size());
			
			for (int i = 0; i < methods.size(); i++) {
				PaymentMethodVO method = methods.get(i);
				log.info("결제수단[" + i + "] - MethodIdx: " + method.getMethod_idx() + 
						", Provider: " + method.getMethod_provider() + 
						", Card: " + method.getMethod_card());
			}
			
			return methods;
			
		} catch (Exception e) {
			log.error("결제수단 조회 중 오류 발생 - memberIdx: " + memberIdx, e);
			e.printStackTrace();
			throw new RuntimeException("결제수단 조회 실패: " + e.getMessage(), e);
		}
	}
	
	// 빌링키 정보 가져오기
	@Override
	public Object getBillingKeyInfo(int methodIdx) {
		try {
			String billingKey = paymentMethodMapper.selectBillingKeyByMethodIdx(methodIdx).getMethod_key();
			
			HttpResponse<String> response = portOneApiClient.getBillingKeyInfo(billingKey);
			
			if (portOneApiClient.isSuccessResponse(response)) {
				// JSON 응답을 Map으로 파싱
				ObjectMapper objectMapper = new ObjectMapper();
				@SuppressWarnings("unchecked")
				Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
				
				// 카드 정보 추출
				Map<String, Object> cardInfo = extractMethodInfo(responseData);
				
				// 성공 응답 반환
				Map<String, Object> result = new HashMap<>();
				result.put("statusCode", response.statusCode());
				result.put("success", true);
				result.put("data", responseData);
				result.put("cardInfo", cardInfo);  // 추출된 카드 정보 추가
				result.put("message", "빌링키 정보 조회 성공");
				
				return result;
			} else {
				// 실패 응답
				Map<String, Object> result = new HashMap<>();
				result.put("statusCode", response.statusCode());
				result.put("success", false);
				result.put("data", response.body());
				result.put("message", "빌링키 정보 조회 실패");
				return result;
			}
			
		} catch (Exception e) {
			log.error("빌링키 정보 조회 중 오류 발생: ", e);
			Map<String, Object> errorResult = new HashMap<>();
			errorResult.put("success", false);
			errorResult.put("message", "빌링키 정보 조회 실패: " + e.getMessage());
			errorResult.put("error", e.getClass().getSimpleName());
			return errorResult;
		}
	}
	
	/**
	 * PortOne API 응답에서 결제 수단 정보를 추출하는 메서드 (빌링키 조회 & 결제 단건 조회 모두 지원)
	 * @param responseData PortOne API 응답 데이터
	 * @return 추출된 카드 정보 (name, number, publisher, issuer, pgProvider 등)
	 */
	@SuppressWarnings("unchecked")
	private Map<String, Object> extractMethodInfo(Map<String, Object> responseData) {
		Map<String, Object> methodInfo = new HashMap<>();
		
		try {
			Map<String, Object> card = null;
			String methodType = null;
			
			// 1. 결제 단건 조회 응답 구조 체크 (method 객체)
			Map<String, Object> method = (Map<String, Object>) responseData.get("method");
			if (method != null) {
				methodType = (String) method.get("type");
				if ("PaymentMethodCard".equals(methodType)) {
					card = (Map<String, Object>) method.get("card");
					log.info("결제 단건 조회 응답에서 카드 정보 추출 시도");
				}
			}
			
			// 2. 빌링키 조회 응답 구조 체크 (methods 배열)
			if (card == null) {
				List<Map<String, Object>> methods = (List<Map<String, Object>>) responseData.get("methods");
				List<Map<String, Object>> channels = (List<Map<String, Object>>) responseData.get("channels");
				if (methods != null && !methods.isEmpty()) {
					// 첫번째 요소 가져오기
					Map<String, Object> firstMethod = methods.get(0);
					Map<String, Object> firstChannel = channels.get(0);

					methodType = (String) firstMethod.get("type");
					// 카드 결제
					if ("BillingKeyPaymentMethodCard".equals(methodType)) {
						card = (Map<String, Object>) firstMethod.get("card");
					}
					// 간편결제
					else if ("BillingKeyPaymentMethodEasyPay".equals(methodType)) {
						methodInfo.put("pgProvider", firstChannel.get("pgProvider"));
					} 
					// 알 수 없는 결제 수단 타입
					else {
						log.warn("알 수 없는 결제 수단 타입: " + methodType);
					}
				}
			}
			
			// 3. 카드 정보 추출
			if (card != null) {
				methodInfo.put("name", card.get("name"));           // 카드 이름 (예: "기업은행카드")
				methodInfo.put("number", card.get("number"));       // 카드 번호 (마스킹됨)
				methodInfo.put("publisher", card.get("publisher")); // 발행사
				methodInfo.put("issuer", card.get("issuer"));       // 발급사
				methodInfo.put("brand", card.get("brand"));         // 브랜드
				methodInfo.put("type", card.get("type"));           // 카드 타입 (DEBIT/CREDIT)
				methodInfo.put("bin", card.get("bin"));             // BIN 코드
				
				log.info("카드 정보 추출 성공 - 방식: " + methodType + ", 카드명: " + card.get("name"));
			}

			// 간편결제("PaymentMethodEasyPay")일 경우 카드 정보를 담지 않음
			if ("PaymentMethodEasyPay".equals(methodType)) {
				log.info("간편결제 방식으로 카드 정보가 없습니다.");
				methodInfo.put("name", null);
				methodInfo.put("number", null);
				methodInfo.put("publisher", null);
				methodInfo.put("issuer", null);
			}

			// 결제 수단 타입 저장
			switch (methodType) {
				case "PaymentMethodCard":
					methodInfo.put("methodType", "card");
					break;
				case "PaymentMethodEasyPay":
					methodInfo.put("methodType", "easyPay");
					break;
				case "BillingKeyPaymentMethodCard":
					methodInfo.put("methodType", "card");
					break;
				case "BillingKeyPaymentMethodEasyPay":
					methodInfo.put("methodType", "easyPay");
					break;
				default:
					break;
			}
			
			// 4. 카드 정보가 없는 경우 기본값 설정
			if (methodInfo.isEmpty()) {
				methodInfo.put("name", "알 수 없는 카드");
				methodInfo.put("number", "****-****-****-****");
				methodInfo.put("publisher", "UNKNOWN");
				methodInfo.put("issuer", "UNKNOWN");
				log.warn("카드 정보를 찾을 수 없어 기본값 설정");
			}
			
		} catch (Exception e) {
			log.error("카드 정보 추출 중 오류 발생: ", e);
			methodInfo.put("name", "정보 추출 실패");
			methodInfo.put("number", "****-****-****-****");
			methodInfo.put("error", e.getMessage());
		}
		
		log.info("추출된 카드 정보: " + methodInfo);
		return methodInfo;
	}

	// 채널키 매칭
	public String getChannelKey(String channelType) {
		return portOneApiClient.getChannelKey(channelType);
	}
	
	// 빌링키로 결제 (api key, payment id, billing key == method key, channel key, ordername, amount, currency 
	@Override
	@Transactional
	public Object payBillingKey(String paymentId, int methodIdx, int memberIdx) {
	    PaymentOrderVO order = null;
	    
	    try {
	        // 1. 결제수단 정보 조회
			PaymentMethodVO method = paymentMethodMapper.selectByMethodIdx(methodIdx);
			if (method == null) {
			    log.error("결제수단을 찾을 수 없습니다. methodIdx: " + methodIdx);
			    Map<String, Object> errorResult = new HashMap<>();
			    errorResult.put("success", false);
			    errorResult.put("message", "결제수단을 찾을 수 없습니다.");
			    return errorResult;
			}
			
			String billingKey = method.getMethod_key();
			String channelKey = getChannelKey(method.getMethod_provider());
			
			log.info("결제 시작 - PaymentId: " + paymentId + ", BillingKey: " + billingKey + ", ChannelKey: " + channelKey);

			// 2. 결제 주문 정보 사전 저장 (READY 상태)
			order = new PaymentOrderVO();
			order.setMember_idx(memberIdx);
			order.setMethod_idx(methodIdx);
			order.setPayment_id(paymentId);
			order.setOrder_type("DIRECT");
			order.setOrder_status("READY");
			order.setOrder_name("FitSync Premium");
			order.setOrder_price(subscribeCost);
			order.setOrder_currency("KRW");
			order.setOrder_regdate(new java.sql.Date(System.currentTimeMillis()));
			
			order.setOrder_provider(method.getMethod_provider());
			String card = method.getMethod_card();
			if (card != null) {
				order.setOrder_card(card);
			}
			String cardNum = method.getMethod_card_num();
			if (cardNum != null) {
				order.setOrder_card_num(cardNum);
			}

			try {
			    paymentOrderMapper.insertPaymentOrder(order);
			    log.info("결제 주문 정보 저장 완료 - PaymentId: " + paymentId);
			    log.info("생성된 order_idx: " + order.getOrder_idx());
			    
			    if (order.getOrder_idx() <= 0) {
			        log.error("order_idx가 생성되지 않았습니다: " + order.getOrder_idx());
			    }
			    
			} catch (Exception dbEx) {
			    log.error("결제 주문 정보 저장 실패: ", dbEx);
			    Map<String, Object> errorResult = new HashMap<>();
			    errorResult.put("success", false);
			    errorResult.put("message", "결제 주문 정보 저장 실패: " + dbEx.getMessage());
			    return errorResult;
			}

			// 3. PortOne API 호출
			HttpResponse<String> response = portOneApiClient.payWithBillingKey(
				paymentId, billingKey, channelKey, "fitsync 구독", subscribeCost
			);
	    		
	    	// 4. 응답 처리 및 주문 상태 업데이트
	    	boolean isSuccess = portOneApiClient.isSuccessResponse(response);
	    	String orderStatus = isSuccess ? "PAID" : "FAILED";
	    	
	    	log.info("=== 결제 상태 업데이트 시작 ===");
	    	log.info("결제 결과 - isSuccess: " + isSuccess + ", orderStatus: " + orderStatus);
	    	log.info("업데이트할 주문 정보 - payment_id: " + paymentId + ", order_idx: " + order.getOrder_idx());
	    	
	    	// 주문 상태 업데이트
	    	order.setOrder_status(orderStatus);
	    	if (isSuccess) {
	    	    order.setOrder_paydate(new java.sql.Date(System.currentTimeMillis()));
	    	    log.info("결제 성공 - 결제일시 설정: " + order.getOrder_paydate());
	    	}
	    	
	    	log.info("업데이트 직전 order 객체 전체: " + order);
	    	
	    	try {
				log.info("결제 주문 정보 업데이트 - " + order);
	    	    paymentOrderMapper.updatePaymentStatus(order);
	    	    log.info("결제 상태 업데이트 완료 - PaymentId: " + paymentId + ", Status: " + orderStatus);
	    	    
	    	    // 업데이트 후 실제 DB 상태 확인
	    	    try {
	    	        PaymentOrderVO updatedOrder = paymentOrderMapper.selectByPaymentId(paymentId);
	    	        if (updatedOrder != null) {
	    	            log.info("업데이트 후 DB 상태: " + updatedOrder);
	    	            if (!"PAID".equals(updatedOrder.getOrder_status()) && isSuccess) {
	    	                log.error("업데이트가 반영되지 않았습니다! 예상: PAID, 실제: " + updatedOrder.getOrder_status());
	    	            }
	    	        } else {
	    	            log.error("업데이트 후 주문을 찾을 수 없습니다!");
	    	        }
	    	    } catch (Exception selectEx) {
	    	        log.error("업데이트 후 조회 실패: ", selectEx);
	    	    }
	    	    
	    	    // 🎯 단건 결제 성공 시 다음 달 자동 결제 예약
	    	    if (isSuccess && "DIRECT".equals(order.getOrder_type())) {
	    	        try {
	    	            log.info("🎯 단건 결제 성공 - 다음 달 자동 결제 예약 시작");
	    	            Object autoScheduleResult = scheduleNextMonthPayment(order);
	    	            
	    	            @SuppressWarnings("unchecked")
	    	            Map<String, Object> scheduleResult = (Map<String, Object>) autoScheduleResult;
	    	            boolean autoSuccess = (boolean) scheduleResult.get("success");
	    	            
	    	            if (autoSuccess) {
	    	                log.info("✅ 단건 결제 후 다음 달 자동 예약 성공 - PaymentId: " + paymentId + 
	    	                        ", NextPaymentId: " + scheduleResult.get("paymentId"));
	    	            } else {
	    	                log.warn("⚠️ 단건 결제 후 다음 달 자동 예약 실패 - PaymentId: " + paymentId + 
	    	                        ", Reason: " + scheduleResult.get("message"));
	    	            }
	    	        } catch (Exception autoEx) {
	    	            log.error("❌ 단건 결제 후 자동 예약 중 예외 발생 - PaymentId: " + paymentId, autoEx);
	    	        }
	    	    }
	    	    
	    	} catch (Exception updateEx) {
	    	    log.error("결제 상태 업데이트 실패: ", updateEx);
				System.out.println("업데이트 중 오류 발생함." + updateEx.getMessage());
	    	    updateEx.printStackTrace();
	    	    // 결제는 성공했지만 상태 업데이트 실패한 경우 별도 처리 필요
	    	}
	    		
	    	// 5. JSON 응답 파싱 및 결과 반환
	    	try {
	    		ObjectMapper objectMapper = new ObjectMapper();
	    		Object responseData = objectMapper.readValue(response.body(), Object.class);
	    			
	    		Map<String, Object> result = new HashMap<>();
	    		result.put("statusCode", response.statusCode());
	    		result.put("success", isSuccess);
	    		result.put("data", responseData);
	    		result.put("message", isSuccess ? "Payment successful" : "Payment failed");
	    		result.put("paymentId", paymentId);
	    		result.put("orderStatus", orderStatus);
	    			
	    		return result;
	    		
	    	} catch (Exception jsonEx) {
	    		log.error("JSON 파싱 실패: ", jsonEx);
	    		Map<String, Object> result = new HashMap<>();
	    		result.put("statusCode", response.statusCode());
	    		result.put("success", false);
	    		result.put("data", response.body());
	    		result.put("message", "Failed to parse response");
	    		result.put("paymentId", paymentId);
	    		return result;
	    	}
	    		
	    } catch (IOException | InterruptedException e) {
	        log.error("PortOne API 호출 실패: ", e);
	        
	        // API 호출 실패 시 주문 상태를 FAILED로 업데이트
	        if (order != null) {
	            try {
	                order.setOrder_status("FAILED");
	                paymentOrderMapper.updatePaymentStatus(order);
	                log.info("API 실패로 인한 주문 상태 업데이트 완료 - PaymentId: " + paymentId);
	            } catch (Exception updateEx) {
	                log.error("API 실패 후 주문 상태 업데이트 실패: ", updateEx);
	            }
	        }
	        
	        Map<String, Object> errorResult = new HashMap<>();
	        errorResult.put("success", false);
	        errorResult.put("message", "Request failed: " + e.getMessage());
	        errorResult.put("error", e.getClass().getSimpleName());
	        errorResult.put("paymentId", paymentId);
	        return errorResult;
	    } catch (Exception e) {
	        log.error("예상치 못한 오류 발생: ", e);
	        
	        // 예상치 못한 오류 시 주문 상태를 FAILED로 업데이트
	        if (order != null) {
	            try {
	                order.setOrder_status("FAILED");
	                paymentOrderMapper.updatePaymentStatus(order);
	            } catch (Exception updateEx) {
	                log.error("예외 발생 후 주문 상태 업데이트 실패: ", updateEx);
	            }
	        }
	        
	        Map<String, Object> errorResult = new HashMap<>();
	        errorResult.put("success", false);
	        errorResult.put("message", "Unexpected error: " + e.getMessage());
	        errorResult.put("error", e.getClass().getSimpleName());
	        errorResult.put("paymentId", paymentId);
	        return errorResult;
	    }
	}

	// 빌링키 결제 예약
	@Override
	@Transactional
	public Object scheduleBillingKey(String paymentId, int methodIdx, int memberIdx, String scheduleDateTime) {
		log.info("=== 결제 예약 시작 ===");
		log.info("PaymentId: " + paymentId + ", MethodIdx: " + methodIdx + ", MemberIdx: " + memberIdx + ", ScheduleDateTime: " + scheduleDateTime);
		
		try {
			// 1. 결제수단 정보 조회
			PaymentMethodVO method = paymentMethodMapper.selectByMethodIdx(methodIdx);
			if (method == null) {
				log.error("결제수단을 찾을 수 없습니다. methodIdx: " + methodIdx);
				return createErrorResponse("결제수단을 찾을 수 없습니다.", paymentId);
			}
			
			String billingKey = method.getMethod_key();
			String channelKey = getChannelKey(method.getMethod_provider());
			
			log.info("결제수단 정보 - BillingKey: " + billingKey + ", Provider: " + method.getMethod_provider() + ", ChannelKey: " + channelKey);
			
			// 2. 날짜/시간 처리 및 유효성 검증
			String apiTimeToPay = processScheduleDateTime(scheduleDateTime);
			if (apiTimeToPay == null) {
				return createErrorResponse("잘못된 날짜 형식이거나 시간이 유효하지 않습니다.", paymentId);
			}
			
			// 3. PortOne API 호출 먼저 실행
			log.info("=== PortOne API 호출 시작 ===");
			HttpResponse<String> response = portOneApiClient.createPaymentSchedule(
				paymentId, billingKey, channelKey, "FitSync Premium", subscribeCost, apiTimeToPay
			);
			
			// 4. API 응답 처리
			if (portOneApiClient.isSuccessResponse(response)) {
				String scheduleId = extractScheduleId(response.body());
				
				if (scheduleId != null) {
					log.info("PortOne API 성공 - ScheduleId: " + scheduleId);
					
					// 5. DB에 모든 정보를 한 번에 저장 (schedule_id 포함)
					PaymentOrderVO order = createScheduleOrder(paymentId, methodIdx, memberIdx, method, scheduleDateTime, scheduleId);
					
					log.info("=== DB 저장 시작 ===");
					log.info("저장할 주문 정보: " + order.toString());
					
					try {
						paymentOrderMapper.insertPaymentOrder(order);
						log.info("DB Insert 완료 - Auto-generated OrderIdx: " + order.getOrder_idx());
					} catch (Exception dbEx) {
						log.error("DB Insert 실패: ", dbEx);
						dbEx.printStackTrace();
						
						// DB 저장 실패 시 PortOne 예약도 취소 시도
						try {
							portOneApiClient.cancelPaymentSchedule(scheduleId);
							log.info("DB 저장 실패로 인한 PortOne 예약 취소 완료");
						} catch (Exception cancelEx) {
							log.error("PortOne 예약 취소 실패: ", cancelEx);
						}
						
						return createErrorResponse("DB 저장 실패: " + dbEx.getMessage(), paymentId);
					}
					
					if (order.getOrder_idx() == 0) {
						log.error("주문 정보 저장 실패 - OrderIdx가 0입니다");
						
						// OrderIdx 생성 실패 시 PortOne 예약도 취소 시도
						try {
							portOneApiClient.cancelPaymentSchedule(scheduleId);
							log.info("OrderIdx 생성 실패로 인한 PortOne 예약 취소 완료");
						} catch (Exception cancelEx) {
							log.error("PortOne 예약 취소 실패: ", cancelEx);
						}
						
						return createErrorResponse("주문 정보 저장에 실패했습니다.", paymentId);
					}
					
					log.info("주문 정보 저장 완료 - OrderIdx: " + order.getOrder_idx());
					
					// 저장 후 실제 DB에서 조회해서 확인
					try {
						PaymentOrderVO savedOrder = paymentOrderMapper.selectByOrderIdx(order.getOrder_idx());
						if (savedOrder != null) {
							log.info("DB 저장 검증 성공 - 저장된 데이터: PaymentId=" + savedOrder.getPayment_id() + 
									", Status=" + savedOrder.getOrder_status() + ", Type=" + savedOrder.getOrder_type() +
									", ScheduleId=" + savedOrder.getSchedule_id() + ", ScheduleDate=" + savedOrder.getSchedule_date());
						} else {
							log.warn("DB 저장 검증 실패 - 저장된 데이터를 찾을 수 없습니다");
						}
					} catch (Exception verifyEx) {
						log.warn("DB 저장 검증 중 오류: " + verifyEx.getMessage());
					}
					
					log.info("결제 예약 성공 - ScheduleId: " + scheduleId);
					return createSuccessResponse(paymentId, scheduleId, scheduleDateTime, order.getOrder_idx());
					
				} else {
					log.error("schedule_id 추출 실패 - API Response: " + response.body());
					return createErrorResponse("예약 등록에 실패했습니다. (schedule_id 추출 실패)", paymentId);
				}
			} else {
				log.error("PortOne API 호출 실패 - Status: " + response.statusCode() + ", Body: " + response.body());
				return createErrorResponse("결제 예약 API 호출에 실패했습니다.", paymentId);
			}
			
		} catch (Exception e) {
			log.error("결제 예약 중 예외 발생: ", e);
			return createErrorResponse("결제 예약 처리 중 오류가 발생했습니다: " + e.getMessage(), paymentId);
		}
	}
	
	/**
	 * 스케줄 날짜/시간 처리 및 유효성 검증
	 */
	private String processScheduleDateTime(String scheduleDateTime) {
		try {
			// 한국 시간대 설정
			java.time.ZoneId koreaZone = java.time.ZoneId.of("Asia/Seoul");
			java.time.LocalDateTime scheduleTime;
			
			// 입력 형식 처리: "yyyy-MM-dd HH:mm:ss" 또는 "yyyy-MM-ddTHH:mm:ss"
			if (scheduleDateTime.contains("T")) {
				scheduleTime = java.time.LocalDateTime.parse(scheduleDateTime);
			} else {
				scheduleTime = java.time.LocalDateTime.parse(scheduleDateTime.replace(" ", "T"));
			}
			
			// 한국 시간대로 변환
			java.time.ZonedDateTime koreaZonedTime = scheduleTime.atZone(koreaZone);
			
			// 현재 시간과 비교하여 유효성 검사
			java.time.ZonedDateTime nowKorea = java.time.ZonedDateTime.now(koreaZone);
			if (koreaZonedTime.isBefore(nowKorea) || koreaZonedTime.isEqual(nowKorea)) {
				log.error("예약 시간이 현재 시간보다 이전입니다 - 현재: " + nowKorea + ", 예약: " + koreaZonedTime);
				return null;
			}
			
			// PortOne API 형식으로 변환 (ISO 8601 형식)
			String apiTimeToPay = koreaZonedTime.format(java.time.format.DateTimeFormatter.ISO_OFFSET_DATE_TIME);
			
			log.info("시간 처리 완료 - 입력: " + scheduleDateTime + ", API 형식: " + apiTimeToPay);
			return apiTimeToPay;
			
		} catch (Exception e) {
			log.error("날짜 형식 오류 - 입력값: " + scheduleDateTime, e);
			return null;
		}
	}
	
	/**
	 * 스케줄 주문 정보 생성 (schedule_id 포함)
	 */
	private PaymentOrderVO createScheduleOrder(String paymentId, int methodIdx, int memberIdx, PaymentMethodVO method, String scheduleDateTime, String scheduleId) {
		PaymentOrderVO order = new PaymentOrderVO();
		order.setMember_idx(memberIdx);
		order.setMethod_idx(methodIdx);
		order.setPayment_id(paymentId);
		order.setOrder_type("SCHEDULE");
		order.setOrder_status("READY"); // 초기 상태
		order.setOrder_name("FitSync Premium");
		order.setOrder_price(subscribeCost);
		order.setOrder_currency("KRW");
		order.setOrder_regdate(new java.sql.Date(System.currentTimeMillis()));
		order.setOrder_provider(method.getMethod_provider());
		
		// PortOne API에서 받은 schedule_id 설정
		order.setSchedule_id(scheduleId);
		
		// 카드 정보 설정
		if (method.getMethod_card() != null) {
			order.setOrder_card(method.getMethod_card());
		}
		if (method.getMethod_card_num() != null) {
			order.setOrder_card_num(method.getMethod_card_num());
		}
		
		// 스케줄 날짜 설정
		try {
			java.time.LocalDateTime scheduleTime;
			if (scheduleDateTime.contains("T")) {
				scheduleTime = java.time.LocalDateTime.parse(scheduleDateTime);
			} else {
				scheduleTime = java.time.LocalDateTime.parse(scheduleDateTime.replace(" ", "T"));
			}
			order.setSchedule_date(java.sql.Timestamp.valueOf(scheduleTime));
			log.info("스케줄 날짜 설정 완료: " + order.getSchedule_date());
		} catch (Exception e) {
			log.error("스케줄 날짜 설정 실패: " + e.getMessage(), e);
		}
		
		log.info("스케줄 주문 정보 생성 완료 - PaymentId: " + paymentId + ", ScheduleId: " + scheduleId + ", ScheduleDate: " + order.getSchedule_date());
		return order;
	}
	
	/**
	 * API 응답에서 schedule_id 추출
	 */
	private String extractScheduleId(String responseBody) {
		try {
			ObjectMapper objectMapper = new ObjectMapper();
			@SuppressWarnings("unchecked")
			Map<String, Object> responseData = objectMapper.readValue(responseBody, Map.class);
			
			// PortOne API v2 응답 구조에 따라 schedule_id 추출
			// 응답 구조: {"schedule": {"id": "schedule_id"}} 또는 {"id": "schedule_id"}
			Object scheduleObj = responseData.get("schedule");
			if (scheduleObj instanceof Map) {
				@SuppressWarnings("unchecked")
				Map<String, Object> schedule = (Map<String, Object>) scheduleObj;
				return (String) schedule.get("id");
			}
			
			// 직접 id 필드가 있는 경우
			return (String) responseData.get("id");
			
		} catch (Exception e) {
			log.error("schedule_id 추출 중 오류 발생: ", e);
			return null;
		}
	}
	
	/**
	 * 성공 응답 생성
	 */
	private Map<String, Object> createSuccessResponse(String paymentId, String scheduleId, String scheduleDateTime, int orderIdx) {
		Map<String, Object> result = new HashMap<>();
		result.put("success", true);
		result.put("message", "결제 예약이 성공적으로 등록되었습니다.");
		result.put("paymentId", paymentId);
		result.put("scheduleId", scheduleId);
		result.put("scheduleDateTime", scheduleDateTime);
		result.put("orderIdx", orderIdx);
		return result;
	}
	
	/**
	 * 오류 응답 생성
	 */
	private Map<String, Object> createErrorResponse(String message, String paymentId) {
		Map<String, Object> result = new HashMap<>();
		result.put("success", false);
		result.put("message", message);
		result.put("paymentId", paymentId);
		return result;
	}

	// 빌링키 결제 예약 취소 (단건)
	@Override
	public Object cancelScheduledPayment(int orderIdx, int memberIdx) {
		try {
			// 예약 취소를 위해 order_idx로 schedule_id 조회
			PaymentOrderVO order = paymentOrderMapper.selectPaymentOrderById(orderIdx);
			if (order == null) {
				log.error("예약 취소 실패 - order_idx: " + orderIdx + "에 해당하는 주문이 없습니다.");
				return Map.of("success", false, "message", "주문을 찾을 수 없습니다.");
			}
			
			String scheduleId = order.getSchedule_id();
			if (scheduleId == null) {
				log.error("예약 취소 실패 - schedule_id가 없습니다. order_idx: " + orderIdx);
				return Map.of("success", false, "message", "예약 정보를 찾을 수 없습니다.");
			}
			
			log.info("단건 예약 취소 시작 - orderIdx: " + orderIdx + ", scheduleId: " + scheduleId);

			// PortOne API로 예약 취소
			HttpResponse<String> response = portOneApiClient.cancelPaymentSchedule(scheduleId);
			log.info("예약 취소 API 응답: Status=" + response.statusCode() + ", Body=" + response.body());

			// API 호출 성공 시 DB 상태 업데이트
			if (portOneApiClient.isSuccessResponse(response)) {
				// API 응답 파싱하여 실제 취소된 스케줄 ID 확인
				ObjectMapper objectMapper = new ObjectMapper();
				@SuppressWarnings("unchecked")
				Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
				
				@SuppressWarnings("unchecked")
				List<String> revokedScheduleIds = (List<String>) responseData.get("revokedScheduleIds");
				
				if (revokedScheduleIds != null && revokedScheduleIds.contains(scheduleId)) {
					// 예약 상태를 CANCELLED로 업데이트
					order.setOrder_status("CANCELLED");
					paymentOrderMapper.updatePaymentStatus(order);
					
					log.info("단건 예약 취소 성공 - ScheduleId: " + scheduleId + " -> CANCELLED");
					return Map.of("success", true, "message", "예약이 성공적으로 취소되었습니다.", "orderIdx", orderIdx, "scheduleId", scheduleId);
				} else {
					log.warn("API 응답에서 해당 스케줄 ID를 찾을 수 없습니다: " + scheduleId);
					return Map.of("success", false, "message", "예약 취소 확인 실패");
				}
			} else {
				log.error("예약 취소 API 실패 - Status: " + response.statusCode() + ", Body: " + response.body());
				return Map.of("success", false, "message", "예약 취소에 실패했습니다. 상태 코드: " + response.statusCode());
			}
		} catch (Exception e) {
			log.error("예약 취소 중 오류 발생 - orderIdx: " + orderIdx, e);
			return Map.of("success", false, "message", "예약 취소 처리 중 오류가 발생했습니다: " + e.getMessage());
		}
	}
	
	// 결제수단명 변경
	@Override
	public boolean renameBillingKey(int memberIdx, int methodIdx, String methodName) {
		try {
			// VO 객체 생성하여 파라미터 전달
			PaymentMethodVO vo = new PaymentMethodVO();
			vo.setMember_idx(memberIdx);
			vo.setMethod_idx(methodIdx);
			vo.setMethod_name(methodName);
			
			int updatedRows = paymentMethodMapper.updatePaymentMethodNameSecure(vo);
			return updatedRows > 0;
		} catch (Exception e) {
			e.printStackTrace();
			return false;
		}
	}
	
	// 결제수단별 모든 예약 취소 (내부 메서드, 리스트 불러올 필요 없음 추후 수정)
	private void cancelAllSchedulesByMethodIdx(int methodIdx) throws Exception {
		try {
			// 해당 결제수단의 모든 예약 조회
			List<PaymentOrderVO> scheduledPayments = paymentOrderMapper.selectScheduledPaymentsByMethodIdx(methodIdx);
			
			if (scheduledPayments.isEmpty()) {
				log.info("취소할 예약이 없습니다. methodIdx: " + methodIdx);
				return;
			}
			
			// 빌링키 조회
			PaymentMethodVO paymentMethod = paymentMethodMapper.selectBillingKeyByMethodIdx(methodIdx);
			if (paymentMethod == null) {
				throw new RuntimeException("결제수단을 찾을 수 없습니다. methodIdx: " + methodIdx);
			}
			
			String billingKey = paymentMethod.getMethod_key();
			log.info("빌링키로 모든 예약 취소 시작 - billingKey: " + billingKey + ", 예약 건수: " + scheduledPayments.size());
			
			// PortOne API로 빌링키의 모든 예약 취소
			HttpResponse<String> response = portOneApiClient.cancelScheduleByBillingKey(billingKey);
			log.info("예약 취소 API 응답: Status=" + response.statusCode() + ", Body=" + response.body());
			
			if (!portOneApiClient.isSuccessResponse(response)) {
				throw new RuntimeException("예약 취소 API 호출 실패. Status: " + response.statusCode() + ", Body: " + response.body());
			}
			
			// API 응답 파싱하여 취소된 스케줄 ID 확인
			ObjectMapper objectMapper = new ObjectMapper();
			@SuppressWarnings("unchecked")
			Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
			
			@SuppressWarnings("unchecked")
			List<String> revokedScheduleIds = (List<String>) responseData.get("revokedScheduleIds");
			
			if (revokedScheduleIds != null && !revokedScheduleIds.isEmpty()) {
				// DB에서 해당 예약들의 상태를 CANCELLED로 업데이트
				for (PaymentOrderVO scheduledPayment : scheduledPayments) {
					String scheduleId = scheduledPayment.getSchedule_id();
					if (scheduleId != null && revokedScheduleIds.contains(scheduleId)) {
						scheduledPayment.setOrder_status("CANCELLED");
						paymentOrderMapper.updatePaymentStatus(scheduledPayment);
						log.info("예약 상태 업데이트 완료 - scheduleId: " + scheduleId + " -> CANCELLED");
					}
				}
				log.info("총 " + revokedScheduleIds.size() + "개의 예약이 취소되었습니다.");
			} else {
				log.warn("API 응답에서 취소된 스케줄 ID를 찾을 수 없습니다.");
			}
			
		} catch (Exception e) {
			log.error("예약 취소 중 오류 발생: ", e);
			throw new RuntimeException("예약 취소 실패: " + e.getMessage(), e);
		}
	}

	// 결제수단 및 빌링키 삭제 (트랜잭션 필요)
	@Transactional(rollbackFor = Exception.class) // 모든 Exception에 대해 롤백
	@Override
	public boolean deletePaymentMethod(int memberIdx, int methodIdx) {
		try {
			log.info("결제수단 삭제 시작 - memberIdx: " + memberIdx + ", methodIdx: " + methodIdx);
			
			// 1. 먼저 해당 결제수단의 모든 예약 취소 (PortOne API + DB 업데이트)
			cancelAllSchedulesByMethodIdx(methodIdx);
			
			// 2. 빌링키 조회
			PaymentMethodVO paymentMethod = paymentMethodMapper.selectBillingKeyByMethodIdx(methodIdx);
			if (paymentMethod == null) {
				throw new RuntimeException("결제수단을 찾을 수 없습니다. methodIdx: " + methodIdx);
			}
			
			String billingKey = paymentMethod.getMethod_key();
			log.info("빌링키 삭제 시작 - billingKey: " + billingKey);
			
			// 3. PortOne API로 빌링키 삭제
			if (billingKey != null && !billingKey.isEmpty()) {
				HttpResponse<String> response = portOneApiClient.deleteBillingKey(billingKey);
				log.info("빌링키 삭제 API 응답: Status=" + response.statusCode() + ", Body=" + response.body());
				
				// 빌링키 삭제 실패시 예외 발생으로 트랜잭션 롤백
				if (!portOneApiClient.isSuccessResponse(response)) {
					throw new RuntimeException("빌링키 삭제에 실패했습니다. 상태 코드: " + response.statusCode() + ", 응답: " + response.body());
				}
				
				log.info("빌링키 삭제 성공 - billingKey: " + billingKey);
			}
			
			// 4. DB에서 결제수단 삭제
			PaymentMethodVO vo = new PaymentMethodVO();
			vo.setMember_idx(memberIdx);
			vo.setMethod_idx(methodIdx);
			
			int deletedRows = paymentMethodMapper.deletePaymentMethod(vo);
			log.info("DB 결제수단 삭제 결과: " + deletedRows + "건");
			
			if (deletedRows == 0) {
				throw new RuntimeException("DB에서 결제수단 삭제 실패. 해당 결제수단이 존재하지 않거나 권한이 없습니다.");
			}
			
			log.info("결제수단 삭제 완료 - memberIdx: " + memberIdx + ", methodIdx: " + methodIdx);
			return true;
			
		} catch (Exception e) {
			log.error("결제수단 삭제 중 오류 발생 - memberIdx: " + memberIdx + ", methodIdx: " + methodIdx, e);
			// RuntimeException을 다시 throw하여 트랜잭션 롤백 발생
			throw new RuntimeException("결제수단 삭제 실패: " + e.getMessage(), e);
		}
	}

	/**
	 * 빌링키로 카드 정보만 조회
	 * @param billingKey 빌링키
	 * @return 카드 정보 (name, number)
	 */
	public Map<String, Object> getCardInfoByBillingKey(String billingKey) {
		try {
			HttpResponse<String> response = portOneApiClient.getBillingKeyInfo(billingKey);
			
			if (portOneApiClient.isSuccessResponse(response)) {
				ObjectMapper objectMapper = new ObjectMapper();
				@SuppressWarnings("unchecked")
				Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
				
				return extractMethodInfo(responseData);
			} else {
				log.error("카드 정보 조회 실패 - Status: " + response.statusCode() + ", Body: " + response.body());
				Map<String, Object> errorInfo = new HashMap<>();
				errorInfo.put("name", "조회 실패");
				errorInfo.put("number", "****-****-****-****");
				errorInfo.put("error", "API 호출 실패");
				return errorInfo;
			}
			
		} catch (Exception e) {
			log.error("카드 정보 조회 중 오류 발생: ", e);
			Map<String, Object> errorInfo = new HashMap<>();
			errorInfo.put("name", "조회 실패");
			errorInfo.put("number", "****-****-****-****");
			errorInfo.put("error", e.getMessage());
			return errorInfo;
		}
	}
	
	/**
	 * 결제수단 등록 전 중복 체크
	 * @param billingKey 빌링키
	 * @param memberIdx 회원 인덱스
	 * @return 중복 체크 결과와 카드 정보
	 */
	@Override
	public Map<String, Object> checkDuplicatePaymentMethod(String billingKey, int memberIdx) {
		Map<String, Object> result = new HashMap<>();
		
		try {
			// 1. 빌링키로 카드 정보 조회
			Map<String, Object> cardInfo = getCardInfoByBillingKey(billingKey);
			
			if (cardInfo.containsKey("error")) {
				result.put("success", false);
				result.put("message", "카드 정보 조회에 실패했습니다.");
				result.put("error", cardInfo.get("error"));
				return result;
			}
			
			String methodType = (String) cardInfo.get("methodType");
			
			// 2. 카드 결제인 경우에만 중복 확인
			if ("card".equals(methodType)) {
				PaymentMethodVO checkVO = new PaymentMethodVO();
				checkVO.setMember_idx(memberIdx);
				checkVO.setMethod_card((String) cardInfo.get("name"));
				checkVO.setMethod_card_num((String) cardInfo.get("number"));
				
				int duplicateCount = paymentMethodMapper.countDuplicateCard(checkVO);
				
				result.put("success", true);
				result.put("cardInfo", cardInfo);
				result.put("isDuplicate", duplicateCount > 0);
				result.put("duplicateCount", duplicateCount);
				
				if (duplicateCount > 0) {
					PaymentMethodVO duplicateMethod = paymentMethodMapper.findDuplicateCard(checkVO);
					result.put("duplicateMethod", duplicateMethod);
					result.put("message", "동일한 카드가 이미 등록되어 있습니다.");
				} else {
					result.put("message", "새로운 카드입니다.");
				}
			} else {
				// 간편결제 등 카드가 아닌 경우는 중복 체크 안함
				result.put("success", true);
				result.put("cardInfo", cardInfo);
				result.put("isDuplicate", false);
				result.put("duplicateCount", 0);
				result.put("message", "새로운 " + methodType + " 결제수단입니다.");
			}
			
			log.info("중복 체크 결과: " + result);
			return result;
			
		} catch (Exception e) {
			log.error("중복 체크 중 오류 발생: ", e);
			result.put("success", false);
			result.put("message", "중복 체크 중 오류가 발생했습니다: " + e.getMessage());
			result.put("error", e.getClass().getSimpleName());
			return result;
		}
	}
	
	/**
	 * 중복 처리 후 결제수단 저장 (기존 삭제 후 새로 등록)
	 * @param vo 새로운 결제수단 정보
	 * @param replaceExisting 기존 결제수단 교체 여부
	 * @return 처리 결과
	 */
	@Override
	public Map<String, Object> saveBillingKeyWithDuplicateHandling(PaymentMethodVO vo, boolean replaceExisting) {
		Map<String, Object> result = new HashMap<>();
		
		try {
			// 1. 빌링키로 카드 정보 조회
			Map<String, Object> cardInfo = getCardInfoByBillingKey(vo.getMethod_key());
			
			if (cardInfo.containsKey("error")) {
				result.put("success", false);
				result.put("message", "카드 정보 조회에 실패했습니다.");
				return result;
			}
			
			String methodType = (String) cardInfo.get("methodType");
			
			// 2. 결제수단 타입에 따라 카드 정보 설정
			if ("card".equals(methodType)) {
				// 카드 결제인 경우에만 카드 정보 설정
				String cardName = (String) cardInfo.get("name");
				String cardNumber = (String) cardInfo.get("number");
				
				vo.setMethod_card(cardName != null ? cardName : "알 수 없는 카드");
				vo.setMethod_card_num(cardNumber != null ? cardNumber : "****-****-****-****");
				
				// 3. 기존 결제수단 교체인 경우 삭제 먼저 처리 (카드인 경우에만)
				if (replaceExisting) {
					PaymentMethodVO duplicateMethod = paymentMethodMapper.findDuplicateCard(vo);
					if (duplicateMethod != null) {
						PaymentMethodVO deleteVO = new PaymentMethodVO();
						deleteVO.setMember_idx(vo.getMember_idx());
						deleteVO.setMethod_idx(duplicateMethod.getMethod_idx());
						
						int deleteResult = paymentMethodMapper.deletePaymentMethod(deleteVO);
						log.info("기존 중복 결제수단 삭제 결과: " + deleteResult);
					}
				}
			} else {
				// 간편결제 등 카드가 아닌 경우 카드 정보 null로 설정
				vo.setMethod_card(null);
				vo.setMethod_card_num(null);
			}
			
			// 4. 새로운 결제수단 등록
			int insertResult = paymentMethodMapper.insertPaymentMethod(vo);
			
			if (insertResult > 0) {
				result.put("success", true);
				result.put("message", replaceExisting ? "기존 결제수단이 새로운 결제수단으로 교체되었습니다." : "새로운 결제수단이 등록되었습니다.");
				result.put("cardInfo", cardInfo);
				result.put("method_idx", vo.getMethod_idx()); // 새로 등록된 결제수단 ID
			} else {
				result.put("success", false);
				result.put("message", "결제수단 등록에 실패했습니다.");
			}
			
		} catch (Exception e) {
			log.error("결제수단 등록/교체 중 오류 발생: ", e);
			result.put("success", false);
			result.put("message", "결제수단 처리 중 오류가 발생했습니다: " + e.getMessage());
			result.put("error", e.getClass().getSimpleName());
		}
		
		return result;
	}

	/**
	 * 사용자별 결제 기록 조회
	 * @param memberIdx 회원 인덱스
	 * @return 결제 기록 리스트 (최신순)
	 */
	@Override
	public List<PaymentOrderVO> getPaymentHistory(int memberIdx) {
		try {
			log.info("=== 결제 기록 조회 시작 ===");
			log.info("Member ID: " + memberIdx);
			
			// DB 연결 및 매퍼 상태 확인
			if (paymentOrderMapper == null) {
				log.error("PaymentOrderMapper가 null입니다!");
				throw new RuntimeException("PaymentOrderMapper가 초기화되지 않았습니다.");
			}
			
			log.info("PaymentOrderMapper 정상 - DB 조회 시작");
			List<PaymentOrderVO> paymentHistory = paymentOrderMapper.selectPaymentOrdersByMember(memberIdx);
			
			if (paymentHistory == null) {
				log.warn("결제 기록이 null로 반환되었습니다.");
				return new ArrayList<>();
			}
			
			log.info("결제 기록 조회 완료 - memberIdx: " + memberIdx + ", 건수: " + paymentHistory.size());
			
			// 각 결제 기록 상세 정보 로깅
			for (int i = 0; i < paymentHistory.size(); i++) {
				PaymentOrderVO order = paymentHistory.get(i);
				log.info("결제기록[" + i + "] - PaymentId: " + order.getPayment_id() + 
						", Type: " + order.getOrder_type() + ", Status: " + order.getOrder_status() + 
						", Amount: " + order.getOrder_price());
			}
			
			return paymentHistory;
			
		} catch (Exception e) {
			log.error("결제 기록 조회 중 오류 발생 - memberIdx: " + memberIdx, e);
			e.printStackTrace(); // 스택 트레이스 출력
			throw new RuntimeException("결제 기록 조회 실패: " + e.getMessage(), e);
		}
	}

	/**
	 * 사용자별 결제 기록 조회 (결제 수단 정보 포함)
	 * @param memberIdx 회원 인덱스
	 * @return 결제 기록 리스트 (최신순, 결제 수단 정보 포함)
	 */
	@Override
	public List<PaymentOrderWithMethodVO> getPaymentHistoryWithMethod(int memberIdx) {
		try {
//			System.out.println("=== 결제 기록 조회 (API) 함수 시작 ===");
			log.info("결제 기록 조회 시작 (API) - memberIdx: " + memberIdx);
			
			// DB에서 기본 결제 주문 정보만 조회 (JOIN 없이)
			List<PaymentOrderWithMethodVO> paymentHistory = paymentOrderMapper.selectPaymentOrdersByMemberWithMethod(memberIdx);
			
//			System.out.println("DB 조회 완료 - 건수: " + paymentHistory.size());
			log.info("결제 기록 조회 완료 (API) - memberIdx: " + memberIdx + ", 건수: " + paymentHistory.size());
			
			// 각 결제에 대해 PortOne API로 결제 수단 정보 조회
			for (PaymentOrderWithMethodVO order : paymentHistory) {
//				System.out.println("처리 중인 결제 - PaymentId: " + order.getPayment_id() + 
//						", OrderType: " + order.getOrder_type() + ", Status: " + order.getOrder_status());

				// 결제 유형에 따라 다른 API 호출
				String orderType = order.getOrder_type();
				String orderStatus = order.getOrder_status();

				try {
					if ("SCHEDULE".equals(orderType) && 
						("READY".equals(orderStatus) || "CANCELLED".equals(orderStatus))) {
						
						// 예약 결제의 경우: schedule_id로 빌링키 조회 후 결제수단 정보 조회
						String scheduleId = order.getSchedule_id();
						if (scheduleId != null) {
//							System.out.println("예약 결제 처리 중 - ScheduleId: " + scheduleId);
							
							// PortOne API에서 예약 정보 조회
							HttpResponse<String> scheduleResponse = portOneApiClient.getPaymentSchedule(scheduleId);
							
//							System.out.println("예약 정보 API 응답 상태: " + scheduleResponse.statusCode());
							
							if (scheduleResponse.statusCode() >= 200 && scheduleResponse.statusCode() < 300) {
								ObjectMapper objectMapper = new ObjectMapper();
								@SuppressWarnings("unchecked")
								Map<String, Object> scheduleData = objectMapper.readValue(scheduleResponse.body(), Map.class);
								
								// billingKey 추출
								String billingKey = (String) scheduleData.get("billingKey");
								if (billingKey != null) {
//									System.out.println("빌링키 조회 성공 - billingKey: " + billingKey);
									
									// 빌링키로 결제수단 정보 조회
									Map<String, Object> cardInfo = getCardInfoByBillingKey(billingKey);
									
									// PaymentOrderWithMethodVO에 API 정보 설정
									String methodType = (String) cardInfo.get("methodType");
									order.setApiMethodType(methodType);
									order.setApiMethodProvider((String) cardInfo.get("provider"));
									order.setApiPgProvider((String) cardInfo.get("pgProvider"));
									
									// 카드 결제인 경우에만 카드 정보 설정
									if ("card".equals(methodType)) {
										order.setApiCardName((String) cardInfo.get("name"));
										order.setApiCardNumber((String) cardInfo.get("number"));
										order.setApiCardPublisher((String) cardInfo.get("publisher"));
										order.setApiCardIssuer((String) cardInfo.get("issuer"));
										order.setApiCardBrand((String) cardInfo.get("brand"));
										order.setApiCardType((String) cardInfo.get("type"));
									} else {
										// 간편결제 등 카드가 아닌 경우 카드 정보 null로 설정
										order.setApiCardName(null);
										order.setApiCardNumber(null);
										order.setApiCardPublisher(null);
										order.setApiCardIssuer(null);
										order.setApiCardBrand(null);
										order.setApiCardType(null);
									}
									
//									System.out.println("예약 결제 정보 업데이트 완료 - ScheduleId: " + scheduleId + 
//											", 결제수단: " + methodType + ", 카드: " + order.getApiCardName());
								} else {
									System.out.println("예약 정보에서 빌링키를 찾을 수 없습니다.");
									setDefaultApiMethodInfo(order);
								}
							} else {
								System.out.println("예약 정보 API 호출 실패 - Status: " + scheduleResponse.statusCode());
								setDefaultApiMethodInfo(order);
							}
						} else {
							System.out.println("Schedule ID가 없습니다.");
							setDefaultApiMethodInfo(order);
						}
						
					} else {
						// 일반 결제의 경우: payment_id로 결제 정보 조회
//						System.out.println("일반 결제 처리 중 - PaymentId: " + order.getPayment_id());
						
						HttpResponse<String> response = portOneApiClient.getPaymentInfo(order.getPayment_id());
//						System.out.println("일반 결제 API 응답 상태: " + response.statusCode());
						
						if (portOneApiClient.isSuccessResponse(response)) {
							// JSON 응답 파싱
							ObjectMapper objectMapper = new ObjectMapper();
							@SuppressWarnings("unchecked")
							Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
							
//							System.out.println("API 응답 파싱 완료");
							
							// 카드 정보 추출 (개선된 extractMethodInfo 함수 사용)
							Map<String, Object> cardInfo = extractMethodInfo(responseData);
							
							// PaymentOrderWithMethodVO에 API 정보 설정
							String methodType = (String) cardInfo.get("methodType");
							order.setApiMethodType(methodType);
							
							// 카드 결제인 경우에만 카드 정보 설정
							if ("card".equals(methodType)) {
								order.setApiCardName((String) cardInfo.get("name"));
								order.setApiCardNumber((String) cardInfo.get("number"));
								order.setApiCardPublisher((String) cardInfo.get("publisher"));
								order.setApiCardIssuer((String) cardInfo.get("issuer"));
								order.setApiCardBrand((String) cardInfo.get("brand"));
								order.setApiCardType((String) cardInfo.get("type"));
							} else {
								// 간편결제 등 카드가 아닌 경우 카드 정보 null로 설정
								order.setApiCardName(null);
								order.setApiCardNumber(null);
								order.setApiCardPublisher(null);
								order.setApiCardIssuer(null);
								order.setApiCardBrand(null);
								order.setApiCardType(null);
							}

							// channel 정보에서 결제 채널 확인
							@SuppressWarnings("unchecked")
							Map<String, Object> channel = (Map<String, Object>) responseData.get("channel");
							if (channel != null) {
								String pgProvider = (String) channel.get("pgProvider");
								order.setApiMethodProvider(pgProvider != null ? pgProvider : "UNKNOWN");
							} else {
								order.setApiMethodProvider("UNKNOWN");
							}
							
//							System.out.println("일반 결제 정보 업데이트 완료 - PaymentId: " + order.getPayment_id() + 
//									", 카드: " + order.getApiCardName() + " " + order.getApiCardNumber());
							
						} else {
							System.out.println("API 호출 실패 - Status: " + response.statusCode());
							log.warn("PortOne API 호출 실패 - PaymentId: " + order.getPayment_id() + 
									", Status: " + response.statusCode());
							// API 호출 실패 시 기본값 설정
							setDefaultApiMethodInfo(order);
						}
					}
					
				} catch (Exception apiEx) {
					System.out.println("API 호출 중 예외: " + apiEx.getMessage());
					apiEx.printStackTrace();
					log.error("PortOne API 호출 중 오류 발생 - PaymentId: " + order.getPayment_id(), apiEx);
					// API 호출 실패 시 기본값 설정
					setDefaultApiMethodInfo(order);
				}
			}
			
//			System.out.println("=== 결제 기록 조회 (API) 함수 완료 ===");
			log.info("결제 기록 조회 및 API 정보 업데이트 완료 - memberIdx: " + memberIdx);
			return paymentHistory;
			
		} catch (Exception e) {
			System.out.println("전체 프로세스 중 예외 발생: " + e.getMessage());
			e.printStackTrace();
			log.error("결제 기록 조회 실패 (API) - memberIdx: " + memberIdx, e);
			throw new RuntimeException("결제 기록 조회 중 오류가 발생했습니다.", e);
		}
	}

	/**
	 * API 정보 조회 실패 시 기본값 설정
	 * @param order 결제 주문 VO
	 */
	private void setDefaultApiMethodInfo(PaymentOrderWithMethodVO order) {
		order.setApiMethodProvider("UNKNOWN");
		order.setApiMethodType("unknown");
		order.setApiCardName("정보 조회 실패");
		order.setApiCardNumber("****-****-****-****");
		order.setApiCardPublisher("UNKNOWN");
		order.setApiCardIssuer("UNKNOWN");
		order.setApiCardBrand("UNKNOWN");
		order.setApiCardType("UNKNOWN");
	}

	// 결제 예약 정보 조회
	@Override
	public PaymentOrderWithMethodVO getScheduledPaymentOrder(int memberIdx) {
		try {
			PaymentOrderWithMethodVO scheduleOrder = paymentOrderMapper.selectScheduledPaymentOrderByMember(memberIdx);
			if (scheduleOrder == null) {
				log.warn("예약된 결제 주문이 없습니다. memberIdx: " + memberIdx);
				return null;
			}
			// schedule_id로 PortOne API에서 결제 예약 정보 조회
			HttpResponse<String> response = portOneApiClient.getPaymentSchedule(scheduleOrder.getSchedule_id());

			if (portOneApiClient.isSuccessResponse(response)) {
				ObjectMapper objectMapper = new ObjectMapper();
				@SuppressWarnings("unchecked")
				Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
				
				// billingKey 추출
				String billingKey = (String) responseData.get("billingKey");
				if (billingKey != null) {
				} else {
					System.out.println("응답에서 billingKey를 찾을 수 없습니다.");
				}
				// billingKey의 카드 정보 추출
				Map<String, Object> cardInfo = getCardInfoByBillingKey(billingKey);
				if (cardInfo != null && !cardInfo.isEmpty()) {
					scheduleOrder.setApiCardName((String) cardInfo.get("name"));
					scheduleOrder.setApiCardNumber((String) cardInfo.get("number"));
					scheduleOrder.setApiMethodType((String) cardInfo.get("methodType"));
					scheduleOrder.setApiMethodProvider((String) cardInfo.get("provider"));
					scheduleOrder.setApiCardPublisher((String) cardInfo.get("publisher"));
					scheduleOrder.setApiCardIssuer((String) cardInfo.get("issuer"));
					scheduleOrder.setApiCardBrand((String) cardInfo.get("brand"));
					scheduleOrder.setApiCardType((String) cardInfo.get("type"));
				} else {
					System.out.println("카드 정보 조회 실패 또는 카드가 아닙니다.");
				}
				
			} else {
				System.out.println("PortOne API 호출 실패 - Status: " + response.statusCode() + ", Body: " + response.body());
			}

			return scheduleOrder;
		} catch (Exception e) {
			e.printStackTrace();
			return null; // TODO: 예외 처리 로직 추가 필요
		}
	}

	// 예약 id로 결제수단 정보 조회
	public Object getPaymentMethodByScheduleId(String scheduleId) {
		try {
			HttpResponse<String> response = portOneApiClient.getPaymentSchedule(scheduleId);
			
			log.info("예약 결제수단 조회 - Status: " + response.statusCode());
			
			if (portOneApiClient.isSuccessResponse(response)) {
				ObjectMapper objectMapper = new ObjectMapper();
				@SuppressWarnings("unchecked")
				Map<String, Object> responseData = objectMapper.readValue(response.body(), Map.class);
				
				return extractMethodInfo(responseData);
			} else {
				log.error("예약 결제수단 조회 실패 - Status: " + response.statusCode() + ", Body: " + response.body());
				Map<String, Object> errorInfo = new HashMap<>();
				errorInfo.put("name", "조회 실패");
				errorInfo.put("number", "****-****-****-****");
				errorInfo.put("error", "API 호출 실패");
				return errorInfo;
			}
			
		} catch (Exception e) {
			log.error("예약 결제수단 조회 중 오류 발생: ", e);
			Map<String, Object> errorInfo = new HashMap<>();
			errorInfo.put("name", "조회 실패");
			errorInfo.put("number", "****-****-****-****");
			errorInfo.put("error", e.getMessage());
			return errorInfo;
		}
	}

	// 예약건 결제수단 변경 (기존 오더 번호, 새로운 결제수단 번호)
	@Override
	@Transactional
	public Map<String, Object> changeSchedulePaymentMethod(int orderIdx, int methodIdx) {
		try {
			log.info("예약 결제수단 변경 시작 - orderIdx: " + orderIdx + ", methodIdx: " + methodIdx);
			
			// 1. 기존 결제 예약 정보 조회
			PaymentOrderVO oldOrder = paymentOrderMapper.selectByOrderIdx(orderIdx);
			if (oldOrder == null) {
				log.error("기존 주문을 찾을 수 없습니다 - orderIdx: " + orderIdx);
				return createErrorResponse("주문을 찾을 수 없습니다.", null);
			}
			
			// 2. 새로운 결제수단 정보 조회
			PaymentMethodVO newMethod = paymentMethodMapper.selectByMethodIdx(methodIdx);
			if (newMethod == null) {
				log.error("새 결제수단을 찾을 수 없습니다 - methodIdx: " + methodIdx);
				return createErrorResponse("결제수단을 찾을 수 없습니다.", null);
			}
			
			String oldScheduleId = oldOrder.getSchedule_id();
			if (oldScheduleId == null) {
				log.error("기존 예약의 schedule_id가 없습니다 - orderIdx: " + orderIdx);
				return createErrorResponse("예약 정보가 올바르지 않습니다.", null);
			}
			
			log.info("기존 예약 정보 - ScheduleId: " + oldScheduleId + ", ScheduleDate: " + oldOrder.getSchedule_date());
			
			// 3. Date → PortOne API 형식 문자열 변환
			String scheduleDateTime = convertDateToPortOneFormat(oldOrder.getSchedule_date());
			if (scheduleDateTime == null) {
				log.error("날짜 변환 실패 - ScheduleDate: " + oldOrder.getSchedule_date());
				return createErrorResponse("예약 날짜 처리 중 오류가 발생했습니다.", null);
			}
			
			log.info("변환된 예약 시간 - Original: " + oldOrder.getSchedule_date() + ", Converted: " + scheduleDateTime);
			
			// 4. 기존 예약 취소
			log.info("기존 예약 취소 시작 - ScheduleId: " + oldScheduleId);
			HttpResponse<String> cancelResponse = portOneApiClient.cancelPaymentSchedule(oldScheduleId);
			
			if (!portOneApiClient.isSuccessResponse(cancelResponse)) {
				log.error("기존 예약 취소 실패 - Status: " + cancelResponse.statusCode() + ", Body: " + cancelResponse.body());
				return createErrorResponse("기존 예약 취소에 실패했습니다.", null);
			}
			
			log.info("기존 예약 취소 성공");
			
			// 5. 새로운 결제수단으로 예약 생성
			String newPaymentId = generatePaymentId();
			String billingKey = newMethod.getMethod_key();
			String channelKey = getChannelKey(newMethod.getMethod_provider());
			
			log.info("새 예약 생성 시작 - PaymentId: " + newPaymentId + ", BillingKey: " + billingKey + 
					", ChannelKey: " + channelKey + ", ScheduleTime: " + scheduleDateTime);
			
			HttpResponse<String> createResponse = portOneApiClient.createPaymentSchedule(
				newPaymentId, billingKey, channelKey, "FitSync Premium", subscribeCost, scheduleDateTime
			);
			
			if (!portOneApiClient.isSuccessResponse(createResponse)) {
				log.error("새 예약 생성 실패 - Status: " + createResponse.statusCode() + ", Body: " + createResponse.body());
				return createErrorResponse("새 예약 생성에 실패했습니다.", newPaymentId);
			}
			
			// 6. 새 schedule_id 추출
			String newScheduleId = extractScheduleId(createResponse.body());
			if (newScheduleId == null) {
				log.error("새 schedule_id 추출 실패 - Response: " + createResponse.body());
				return createErrorResponse("새 예약 등록에 실패했습니다.", newPaymentId);
			}
			
			log.info("새 예약 생성 성공 - NewScheduleId: " + newScheduleId);
			
			// 7. DB 업데이트 - 기존 주문 정보를 새로운 정보로 업데이트
			oldOrder.setPayment_id(newPaymentId);
			oldOrder.setMethod_idx(methodIdx);
			oldOrder.setSchedule_id(newScheduleId);
			oldOrder.setOrder_provider(newMethod.getMethod_provider());
			
			// 카드 정보 업데이트
			if (newMethod.getMethod_card() != null) {
				oldOrder.setOrder_card(newMethod.getMethod_card());
			} else {
				oldOrder.setOrder_card(null);
			}
			if (newMethod.getMethod_card_num() != null) {
				oldOrder.setOrder_card_num(newMethod.getMethod_card_num());
			} else {
				oldOrder.setOrder_card_num(null);
			}
			
			// DB 업데이트
			paymentOrderMapper.updateScheduledPaymentMethod(oldOrder);
			
			log.info("DB 업데이트 완료 - OrderIdx: " + orderIdx + ", NewMethodIdx: " + methodIdx + 
					", NewScheduleId: " + newScheduleId);
			
			// 8. 성공 응답 반환
			Map<String, Object> result = new HashMap<>();
			result.put("success", true);
			result.put("message", "결제수단이 성공적으로 변경되었습니다.");
			result.put("orderIdx", orderIdx);
			result.put("newPaymentId", newPaymentId);
			result.put("newMethodIdx", methodIdx);
			result.put("newScheduleId", newScheduleId);
			result.put("scheduleDateTime", scheduleDateTime);
			result.put("oldScheduleId", oldScheduleId);
			
			return result;
			
		} catch (Exception e) {
			log.error("예약 결제수단 변경 중 오류 발생 - orderIdx: " + orderIdx + ", methodIdx: " + methodIdx, e);
			
			Map<String, Object> errorResult = new HashMap<>();
			errorResult.put("success", false);
			errorResult.put("message", "결제수단 변경 중 오류가 발생했습니다: " + e.getMessage());
			errorResult.put("error", e.getClass().getSimpleName());
			errorResult.put("orderIdx", orderIdx);
			
			return errorResult;
		}
	}

	/**
	 * Date/Timestamp를 PortOne API 형식으로 변환
	 * @param scheduleDate DB의 schedule_date (java.util.Date 또는 java.sql.Timestamp)
	 * @return PortOne API 형식 문자열 (ISO 8601 with timezone)
	 */
	private String convertDateToPortOneFormat(java.util.Date scheduleDate) {
		try {
			if (scheduleDate == null) {
				log.error("scheduleDate가 null입니다.");
				return null;
			}
			
			// 1. Date를 LocalDateTime으로 변환
			java.time.LocalDateTime localDateTime;
			
			if (scheduleDate instanceof java.sql.Timestamp) {
				// Timestamp인 경우
				localDateTime = ((java.sql.Timestamp) scheduleDate).toLocalDateTime();
			} else {
				// 일반 Date인 경우
				localDateTime = scheduleDate.toInstant()
					.atZone(java.time.ZoneId.systemDefault())
					.toLocalDateTime();
			}
			
			// 2. 한국 시간대 적용
			java.time.ZoneId koreaZone = java.time.ZoneId.of("Asia/Seoul");
			java.time.ZonedDateTime koreaZonedTime = localDateTime.atZone(koreaZone);
			
			// 3. PortOne API 형식으로 변환 (ISO 8601 with offset)
			String portOneFormat = koreaZonedTime.format(java.time.format.DateTimeFormatter.ISO_OFFSET_DATE_TIME);
			
			log.info("날짜 변환 성공 - Input: " + scheduleDate + " → Output: " + portOneFormat);
			return portOneFormat;
			
		} catch (Exception e) {
			log.error("날짜 변환 실패 - Input: " + scheduleDate, e);
			return null;
		}
	}

	/**
	 * 다음 달 자동 결제 예약 (정기 결제용)
	 * 결제 성공 시 31일 후 동일한 결제수단으로 자동 예약
	 * @param completedOrder 완료된 결제 주문 정보
	 * @return 예약 결과
	 */
	@Override
	public Map<String, Object> scheduleNextMonthPayment(PaymentOrderVO completedOrder) {
		try {
			log.info("다음 달 자동 결제 예약 시작 - CompletedOrderIdx: " + completedOrder.getOrder_idx() + 
					", MemberIdx: " + completedOrder.getMember_idx() + ", MethodIdx: " + completedOrder.getMethod_idx());
			
			// 1. 결제수단이 여전히 유효한지 확인
			PaymentMethodVO paymentMethod = paymentMethodMapper.selectByMethodIdx(completedOrder.getMethod_idx());
			if (paymentMethod == null) {
				log.warn("결제수단을 찾을 수 없음 - MethodIdx: " + completedOrder.getMethod_idx());
				return Map.of("success", false, "message", "결제수단을 찾을 수 없습니다.");
			}
			
			Date orderPayDate = completedOrder.getOrder_paydate();
			
			java.time.LocalDateTime lastPaymentDateTime = orderPayDate.toInstant()
			        .atZone(ZoneId.systemDefault())
			        .toLocalDateTime();
			
			// 2. 다음 결제일 계산 (31일 후)
			java.time.LocalDateTime nextPaymentDateTime = lastPaymentDateTime
					.plusDays(31)
					.withHour(0)  // 자정으로 고정
					.withMinute(0)
					.withSecond(0)
					.withNano(0);
			
			log.info("다음 결제 예정일: " + nextPaymentDateTime);
			
			// 3. 새로운 PaymentId 생성
			String nextPaymentId = generatePaymentId();
			
			// 4. 다음 달 결제 예약 호출
			String scheduleDateTime = nextPaymentDateTime.format(java.time.format.DateTimeFormatter.ISO_LOCAL_DATE_TIME);
			Object scheduleResult = scheduleBillingKey(
				nextPaymentId, 
				completedOrder.getMethod_idx(), 
				completedOrder.getMember_idx(), 
				scheduleDateTime
			);
			
			// 5. 결과 확인 및 로깅
			@SuppressWarnings("unchecked")
			Map<String, Object> result = (Map<String, Object>) scheduleResult;
			boolean isSuccess = (boolean) result.get("success");
			
			if (isSuccess) {
				log.info("다음 달 자동 결제 예약 성공 - NextPaymentId: " + nextPaymentId + 
						", NextPaymentDate: " + nextPaymentDateTime + ", ScheduleId: " + result.get("scheduleId"));
				System.out.println("🔄 [자동 예약] 다음 달 결제 예약 완료 - MemberIdx: " + completedOrder.getMember_idx() + 
						", 예약일: " + nextPaymentDateTime.toLocalDate());
						
				// 성공 응답에 추가 정보 포함
				result.put("originalOrderIdx", completedOrder.getOrder_idx());
				result.put("nextPaymentDate", nextPaymentDateTime.toString());
				result.put("isAutoScheduled", true);
			} else {
				log.error("다음 달 자동 결제 예약 실패 - " + result.get("message"));
				System.err.println("❌ [자동 예약] 다음 달 결제 예약 실패 - MemberIdx: " + completedOrder.getMember_idx());
			}
			
			return result;
			
		} catch (Exception e) {
			log.error("다음 달 자동 결제 예약 중 오류 발생 - CompletedOrderIdx: " + completedOrder.getOrder_idx(), e);
			
			Map<String, Object> errorResult = new HashMap<>();
			errorResult.put("success", false);
			errorResult.put("message", "다음 달 자동 결제 예약 중 오류가 발생했습니다: " + e.getMessage());
			errorResult.put("error", e.getClass().getSimpleName());
			errorResult.put("originalOrderIdx", completedOrder.getOrder_idx());
			errorResult.put("isAutoScheduled", true);
			
			return errorResult;
		}
	}

	/**
	 * PaymentId 생성 유틸리티 메서드
	 * @return 고유한 PaymentId
	 */
	private String generatePaymentId() {
		return "auto_" + System.currentTimeMillis() + "_" + 
			   java.util.UUID.randomUUID().toString().substring(0, 8);
	}

	/**
	 * 구독자 여부 확인 및 상세 정보 반환
	 * @param memberIdx 회원 인덱스
	 * @return 구독 상태 정보
	 */
	@Override
	public Map<String, Object> checkSubscriptionStatus(int memberIdx) {
		Map<String, Object> result = new HashMap<>();
		
		try {
			log.info("구독자 상태 확인 시작 - memberIdx: " + memberIdx);
			
			// 1. 활성 구독 확인
			PaymentOrderVO activeSubscription = paymentOrderMapper.selectActiveSubscription(memberIdx);
			
			boolean isSubscriber = (activeSubscription != null);
			result.put("isSubscriber", isSubscriber);
			result.put("memberIdx", memberIdx);
			
			if (isSubscriber) {
				// 2. 구독 상세 정보 설정
//				result.put("subscriptionType", activeSubscription.getOrder_type());
//				result.put("subscriptionStatus", activeSubscription.getOrder_status());
				
				// 3. 구독 유효기간 계산
				if ("PAID".equals(activeSubscription.getOrder_status()) && activeSubscription.getOrder_paydate() != null) {
					// 결제 완료된 구독의 경우
					java.util.Date payDate = activeSubscription.getOrder_paydate();
					java.util.Calendar cal = java.util.Calendar.getInstance();
					cal.setTime(payDate);
					cal.add(java.util.Calendar.DAY_OF_MONTH, 31);
					java.util.Date expiryDate = cal.getTime();
					
					result.put("lastPaymentDate", payDate);
//					result.put("subscriptionExpiryDate", expiryDate);
//					result.put("subscriptionDaysLeft", calculateDaysLeft(expiryDate));
					
					log.info("✅ 활성 구독자 - 마지막 결제일: " + payDate + ", 만료일: " + expiryDate);
					
				} else if ("READY".equals(activeSubscription.getOrder_status()) && activeSubscription.getSchedule_date() != null) {
					// 예약 결제 대기 중인 구독의 경우
//					result.put("nextPaymentDate", activeSubscription.getSchedule_date());
//					result.put("scheduleId", activeSubscription.getSchedule_id());
					
					log.info("📅 예약 구독자 - 다음 결제 예정일: " + activeSubscription.getSchedule_date());
				}
				
				// 4. 결제 수단 정보 (있는 경우)
				if (activeSubscription.getMethod_idx() > 0) {
//					result.put("paymentMethodIdx", activeSubscription.getMethod_idx());
				}
				
				// 5. 구독 시작 정보
//				result.put("subscriptionStartDate", activeSubscription.getOrder_regdate());
//				result.put("subscriptionAmount", activeSubscription.getOrder_price());
//				result.put("orderIdx", activeSubscription.getOrder_idx());

				// 사용량 조회
				Map<String, Object> userUseage = apiLogMapper.selectTokenUsageDuringLatestPaidOrder(memberIdx);
				int inputTokens = ((BigDecimal) userUseage.get("INPUT_TOKENS")).intValue();
				int outputTokens = ((BigDecimal) userUseage.get("OUTPUT_TOKENS")).intValue();

				double totalCost = calculateCost(inputTokens, outputTokens);

//				result.put("inputToken", inputTokens);
//				result.put("outputToken", outputTokens);
				result.put("totalCost", totalCost);
				result.put("isLog", true);

				
			} else {
				log.info("❌ 비구독자 - memberIdx: " + memberIdx);
				result.put("message", "현재 유효한 구독이 없습니다.");
				ApiLogVO log = apiLogMapper.selectFirstRoutineLog(memberIdx);
				boolean isLog = log != null ? true : false;
				result.put("isLog", isLog);
			}
			
			// result.put("checkTimestamp", System.currentTimeMillis());
			log.info("구독자 상태 확인 완료 - memberIdx: " + memberIdx + ", isSubscriber: " + isSubscriber);
			
			return result;
			
		} catch (Exception e) {
			log.error("구독자 상태 확인 중 오류 발생 - memberIdx: " + memberIdx, e);
			result.put("isSubscriber", false);
			result.put("error", true);
			result.put("message", "구독 상태 확인 중 오류가 발생했습니다: " + e.getMessage());
			return result;
		}
	}
	
	// 최근 결제완료건 or 최근 결제예약건 (구독 메인 페이지)
	@Override
	public PaymentOrderVO getRecentOrder(int memberIdx) {
		return paymentOrderMapper.selectRecentOrederBymemberIdx(memberIdx);
	}

	/**
	 * 만료일까지 남은 일수 계산
	 * @param expiryDate 만료일
	 * @return 남은 일수 (음수면 만료됨)
	 */
	private int calculateDaysLeft(java.util.Date expiryDate) {
		long currentTime = System.currentTimeMillis();
		long expiryTime = expiryDate.getTime();
		long diffTime = expiryTime - currentTime;
		return (int) (diffTime / (1000 * 60 * 60 * 24));
	}

	// 포트원 예약 취소 api
	private HttpResponse<String> portOneCancelSchedules(String scheduleId) {
		try {
			return portOneApiClient.cancelPaymentSchedule(scheduleId);
		} catch (Exception e) {
			log.error("예약 취소 API 호출 중 오류 발생: ", e);
			return null;
		}
	}

	// GPT-4o 요금 (USD 기준, 2024년 6월 기준)
    private static final double INPUT_COST_PER_1000 = 0.005;   // $5 / 1M tokens
    private static final double OUTPUT_COST_PER_1000 = 0.015;  // $15 / 1M tokens

    /**
     * 예상 비용 계산 (USD 기준)
     * @param inputTokens 입력 토큰 수
     * @param outputTokens 출력 토큰 수
     * @return 총 비용 (소수점 6자리 반올림)
     */
    public static double calculateCost(int inputTokens, int outputTokens) {
        double inputCost = inputTokens * INPUT_COST_PER_1000 / 1000.0;
        double outputCost = outputTokens * OUTPUT_COST_PER_1000 / 1000.0;
        double total = inputCost + outputCost;

        // 소수점 6자리까지 반올림
        return Math.round(total * 1_000_000) / 1_000_000.0;
    }
}


```

### PortOneApiClient

```java
package org.fitsync.service;

import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

import org.fitsync.config.PortOneConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class PortOneApiClient {
    
    private static final Logger log = LoggerFactory.getLogger(PortOneApiClient.class);
    private static final String PORTONE_BASE_URL = "https://api.portone.io";
    
    private final PortOneConfig portOneConfig;
    
    @Autowired
    public PortOneApiClient(PortOneConfig portOneConfig) {
        this.portOneConfig = portOneConfig;
    }
    
    /**
     * 빌링키 정보 조회
     * @param billingKey 빌링키
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> getBillingKeyInfo(String billingKey) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/billing-keys/" + billingKey))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("GET", HttpRequest.BodyPublishers.noBody())
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne getBillingKeyInfo API Response Status: " + response.statusCode());
        log.info("PortOne getBillingKeyInfo API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 빌링키로 결제 실행
     * @param paymentId 결제 ID
     * @param billingKey 빌링키
     * @param channelKey 채널키
     * @param orderName 주문명
     * @param amount 결제 금액
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> payWithBillingKey(String paymentId, String billingKey, String channelKey, 
                                                 String orderName, int amount) throws IOException, InterruptedException {
        String requestBody = String.format(
            "{\"storeId\":\"%s\",\"billingKey\":\"%s\",\"channelKey\":\"%s\",\"orderName\":\"%s\",\"amount\":{\"total\":%d},\"currency\":\"KRW\"}",
            portOneConfig.getStoreId(), billingKey, channelKey, orderName, amount
        );
        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payments/" + paymentId + "/billing-key"))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("POST", HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne payWithBillingKey API Status Code: " + response.statusCode());
        log.info("PortOne payWithBillingKey API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 결제 정보 조회
     * @param paymentId 결제 ID
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> getPaymentInfo(String paymentId) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payments/" + paymentId))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("GET", HttpRequest.BodyPublishers.noBody())
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne getPaymentInfo API Response Status: " + response.statusCode());
        log.info("PortOne getPaymentInfo API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 빌링키 저장
     * @param billingKey 빌링키
     * @param channelKey 채널키
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> saveBillingKey(String billingKey, String channelKey) throws IOException, InterruptedException {
        String requestBody = String.format(
            "{\"storeId\":\"%s\",\"channelKey\":\"%s\",\"billingKey\":\"%s\"}",
            portOneConfig.getStoreId(), channelKey, billingKey
        );
        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/billing-keys"))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("POST", HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne saveBillingKey API Response Status: " + response.statusCode());
        log.info("PortOne saveBillingKey API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 성공 응답 여부 확인
     * @param response HTTP 응답
     * @return 성공 여부
     */
    public boolean isSuccessResponse(HttpResponse<String> response) {
        return response.statusCode() >= 200 && response.statusCode() < 300;
    }
    
    /**
     * 결제 스케줄 생성
     * @param paymentId 결제 ID
     * @param billingKey 빌링키
     * @param channelKey 채널키
     * @param orderName 주문명
     * @param amount 결제 금액
     * @param scheduleDateTime 예약 시간 (ISO 8601 형식)
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> createPaymentSchedule(String paymentId, String billingKey, String channelKey, 
            String orderName, int amount, String scheduleDateTime) throws IOException, InterruptedException {
        String requestBody = String.format(
            "{\"payment\":{\"storeId\":\"%s\",\"billingKey\":\"%s\",\"channelKey\":\"%s\",\"orderName\":\"%s\",\"amount\":{\"total\":%d},\"currency\":\"KRW\"},\"timeToPay\":\"%s\"}",
            portOneConfig.getStoreId(), billingKey, channelKey, orderName, amount, scheduleDateTime
        );
        
        log.info("PortOne 결제 스케줄 생성 요청 Body: " + requestBody);
        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payments/" + paymentId + "/schedule"))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("POST", HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne createPaymentSchedule API Status Code: " + response.statusCode());
        log.info("PortOne createPaymentSchedule API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 결제 스케줄 취소
     * @param scheduleId 스케줄 ID
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> cancelPaymentSchedule(String scheduleId) throws IOException, InterruptedException {
        String requestBody = String.format("{\"scheduleIds\":[\"%s\"]}", scheduleId);
        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payment-schedules"))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("DELETE", HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne cancelPaymentSchedule API Status Code: " + response.statusCode());
        log.info("PortOne cancelPaymentSchedule API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 빌링키로 스케줄 취소
     * @param billingKey 빌링키
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> cancelScheduleByBillingKey(String billingKey) throws IOException, InterruptedException {
        String requestBody = String.format("{\"billingKey\":\"%s\"}", billingKey);
        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payment-schedules"))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("DELETE", HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne cancelScheduleByBillingKey API Status Code: " + response.statusCode());
        log.info("PortOne cancelScheduleByBillingKey API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 결제 스케줄 조회
     * @param scheduleId 스케줄 ID
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> getPaymentSchedule(String scheduleId) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/payment-schedules/" + scheduleId + "?storeId=" + portOneConfig.getStoreId()))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("GET", HttpRequest.BodyPublishers.noBody())
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne getPaymentSchedule API Status Code: " + response.statusCode());
        log.info("PortOne getPaymentSchedule API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 빌링키 삭제
     * @param billingKey 빌링키
     * @return API 응답
     * @throws IOException
     * @throws InterruptedException
     */
    public HttpResponse<String> deleteBillingKey(String billingKey) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(PORTONE_BASE_URL + "/billing-keys/" + billingKey + "?storeId=" + portOneConfig.getStoreId()))
                .header("Content-Type", "application/json")
                .header("Authorization", "PortOne " + portOneConfig.getApiSecretKey())
                .method("DELETE", HttpRequest.BodyPublishers.ofString("{}"))
                .build();
        
        HttpResponse<String> response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
        
        log.info("PortOne deleteBillingKey API Status Code: " + response.statusCode());
        log.info("PortOne deleteBillingKey API Response Body: " + response.body());
        
        return response;
    }
    
    /**
     * 채널키 선택 (결제 수단에 따라)
     * @param paymentMethod 결제 수단
     * @return 채널키
     */
    public String getChannelKey(String paymentMethod) {
        if (paymentMethod == null) {
            return portOneConfig.getChannelKey();
        }
        
        switch (paymentMethod.toLowerCase()) {
            case "kakaopay":
                return portOneConfig.getKakaopayChannelKey();
            case "tosspayments":
                return portOneConfig.getTosspaymentsChannelKey();
            default:
                return portOneConfig.getChannelKey();
        }
    }
}


```