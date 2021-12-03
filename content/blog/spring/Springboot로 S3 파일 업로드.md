---
title: 'Springboot로 S3 파일 업로드하기'
date: 2021-12-03 13:22:00
category: 'AWS'
draft: false
---

이번 포스팅은 스프링에서 AWS S3 파일 업로드하는 방법입니다. 

주로 이미지 파일을 올릴 때 많이 사용되곤 합니다.

# 1. 의존성 추가하기

- build.gradle

```java
implementation 'io.awspring.cloud:spring-cloud-starter-aws:2.3.1'
```

> [awspring/spring-cloud-aws](https://github.com/awspring/spring-cloud-aws)

Spring-Cloud-AWS 의존성을 추가합니다.

# 2. S3 업로드

### 환경변수 설정

```properties
# AWS Account Credentials (AWS 접근 키)
cloud.aws.credentials.accessKey={액세스키}
cloud.aws.credentials.secretKey={액세스 시크릿 키}

# AWS S3 bucket Info (S3 버킷정보)
cloud.aws.s3.bucket={S3 버킷 이름)
cloud.aws.region.static=ap-northeast-2 (S3 버킷 지역)
cloud.aws.stack.auto=false

# file upload max size (파일 업로드 크기 설정)
spring.servlet.multipart.max-file-size=20MB
spring.servlet.multipart.max-request-size=20MB
```

- EC2에서 Spring Cloud 프로젝트를 실행시키면 기본으로 **CloudFormation 구성을 시작**합니다.
- `spring.servlet.multipart.max-file-size`: 파일 하나당 크기
- `spring.servlet.multipart.max-request-size`: 전송하려는 총 파일들의 크기
- 설정한 CloudFormation이 없으면 프로젝트 시작이 안되니, 해당 내용을 사용하지 않도록 `false`를 등록합니다.

### Controller

```java
  @PostMapping("/upload")
  public String uploadFile(
      @RequestParam("category") String category,
      @RequestPart(value = "file") MultipartFile multipartFile) {
    return awsS3Service.uploadFileV1(category, multipartFile);
  }
```

- `@RequestPart` 애너테이션을 이용해서 multipart/form-data 요청을 받습니다.

### Service

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class AwsS3Service {

  private final AmazonS3Client amazonS3Client;

  @Value("${cloud.aws.s3.bucket}")
  private String bucketName;

  public String uploadFileV1(String category, MultipartFile multipartFile) {
    validateFileExists(multipartFile);

    String fileName = CommonUtils.buildFileName(category, multipartFile.getOriginalFilename());

    ObjectMetadata objectMetadata = new ObjectMetadata();
    objectMetadata.setContentType(multipartFile.getContentType());

    try (InputStream inputStream = multipartFile.getInputStream()) {
      amazonS3Client.putObject(new PutObjectRequest(bucketName, fileName, inputStream, objectMetadata)
          .withCannedAcl(CannedAccessControlList.PublicRead));
    } catch (IOException e) {
      throw new FileUploadFailedException();
    }

    return amazonS3Client.getUrl(bucketName, fileName).toString();
  }

	private void validateFileExists(MultipartFile multipartFile) {
    if (multipartFile.isEmpty()) {
      throw new EmptyFileException();
    }
  }
```

- 이전 AWS 클라우드와 달리 액세스 키, 액세스 시크릿을 환경변수로 설정하기만하면 따로 받을 필요가 없습니다.
- `bucketName`: 버킷이름을 받습니다.
- `validateFileExists` : 파일이 들어있는지 확인하는 메서드
- `CannedAccessControlList.PublicRead` : 퍼블릭으로 할 것인지 프라이빗으로 할 건지 선택이 가능합니다.

### GlobalExceptionHandler

```java
  /**
   * 파일 업로드 용량 초과시 발생
   */
  @ExceptionHandler(MaxUploadSizeExceededException.class)
  protected ResponseEntity<ErrorResponse> handleMaxUploadSizeExceededException(
      MaxUploadSizeExceededException e) {
    log.info("handleMaxUploadSizeExceededException", e);

    ErrorResponse response = ErrorResponse.of(ErrorCode.FILE_SIZE_EXCEED);
    return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
  }
```

- 파일 업로드 용량 초과시 발생하는 에러처리입니다.

### CommonUtils

```java
private static final String FILE_EXTENSION_SEPARATOR = ".";

public static String buildFileName(String category, String originalFileName) {
    int fileExtensionIndex = originalFileName.lastIndexOf(FILE_EXTENSION_SEPARATOR);
    String fileExtension = originalFileName.substring(fileExtensionIndex);
    String fileName = originalFileName.substring(0, fileExtensionIndex);
    String now = String.valueOf(System.currentTimeMillis());

    return category + CATEGORY_PREFIX + fileName + TIME_SEPARATOR + now + fileExtension;
  }
```

파일이름 생성할 때 사용하는 Utils
- `fileExtensionIndex` : 파일 확장자 구분선
- `fileExtension` : 파일 확장자
- `fileName` : 파일 이름
- `now` : 파일 업로드 시간

파일이름은 다음처럼 나타납니다.

![image](https://user-images.githubusercontent.com/49144662/144545385-0252b7f1-97ed-4c2e-8aee-9f6a0725d818.png)

## 2.1 S3 테스트

### 1. 성공테스트

![image](https://user-images.githubusercontent.com/49144662/144545166-43594fc0-37a9-44a3-b849-a4939599f416.png)

포스트맨을 이용하여 파일 업로드 테스트를 합니다.

![image](https://user-images.githubusercontent.com/49144662/144545283-a2db2889-dc8c-4245-a633-d9329af6c62b.png)
파일 업로드 성공했습니다.

### 1.1 비어있는 파일 보내기 - 예외처리

![image](https://user-images.githubusercontent.com/49144662/144545983-3111524b-3167-4bbe-9562-d303f5a4e392.png)
![image](https://user-images.githubusercontent.com/49144662/144545459-5b297d40-ad17-49d8-b380-12c0ccde27b3.png)

비어있는 파일 보내기 예외처리 확인!

### 1.2 업로드 용량 크기 예외처리

![image](https://user-images.githubusercontent.com/49144662/144545498-7e868533-c555-4345-9eed-0541dc2f68cd.png)
![image](https://user-images.githubusercontent.com/49144662/144546102-97864819-8753-470c-b9e1-7dc4906c5774.png)
![image](https://user-images.githubusercontent.com/49144662/144545519-8b1d4fba-941e-4866-929d-1001db721c90.png)

업로드 용량 크기 예외처리 확인!

# 3. S3 다운로드

### Controller

```java
	@GetMapping("/download")
  public ResponseEntity<ByteArrayResource> downloadFile(
      @RequestParam("resourcePath") String resourcePath) {
    byte[] data = awsS3Service.downloadFileV1(resourcePath);
    ByteArrayResource resource = new ByteArrayResource(data);
    HttpHeaders headers = buildHeaders(resourcePath, data);

    return ResponseEntity
        .ok()
        .headers(headers)
        .body(resource);
  }

  private HttpHeaders buildHeaders(String resourcePath, byte[] data) {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentLength(data.length);
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
    headers.setContentDisposition(CommonUtils.createContentDisposition(resourcePath));
    return headers;
  }
```

- `buildHeaders`: 헤더 설정

    - `setContentType(MediaType.APPLICATION_OCTET_STREAM)`
        - 전송하는 파일의 종류에 따라 Content-Type을 지정해줍니다.

    - `setContentDisposition(CommonUtils.createContentDisposition(resourcePath)`
        - 다운로드 받았을 때의 보여줄 파일 이름을 넣습니다.

### Service

```java
	public byte[] downloadFileV1(String resourcePath) {
    validateFileExistsAtUrl(resourcePath);

    S3Object s3Object = amazonS3Client.getObject(bucketName, resourcePath);
    S3ObjectInputStream inputStream = s3Object.getObjectContent();
    try {
      return IOUtils.toByteArray(inputStream);
    } catch (IOException e) {
      throw new FileDownloadFailedException();
    }
  }

	private void validateFileExistsAtUrl(String resourcePath) {
    if (!amazonS3Client.doesObjectExist(bucketName, resourcePath)) {
      throw new FileNotFoundException();
    }
  }
```

- `validateFileExistsAtUrl` : Url에 파일이 있는지 확인하는 메서드

### CommonUtils

```java
	private static final String CATEGORY_PREFIX = "/";
	private static final String TIME_SEPARATOR = "_";
  private static final int UNDER_BAR_INDEX = 1;
	
	public static ContentDisposition createContentDisposition(String categoryWithFileName) {
    String fileName = categoryWithFileName.substring(
        categoryWithFileName.lastIndexOf(CATEGORY_PREFIX) + UNDER_BAR_INDEX);
    return ContentDisposition.builder("attachment")
        .filename(fileName, StandardCharsets.UTF_8)
        .build();
  }
```

![image](https://user-images.githubusercontent.com/49144662/144545577-2f595fd5-a253-4dad-9550-db69ae26731b.png)

- `fileName` : 카테고리(Prefix) + 카테고리 다음 언더바까지 잘라 파일이름을 만듭니다.
- UTF_8을 추가하여 한글 파일이름일 경우 깨지는 것을 방지합니다.

# 추가: 다중 업로드

### Controller
```java
  @PostMapping("/upload")
  public FileUploadResponse uploadFile(
      @RequestParam("category") String category,
      @RequestPart(value = "file") List<MultipartFile> multipartFiles,
      Authentication authentication) {
    long userId = userService.retrieveUserIdByUsername(authentication.getName());
    log.info("upload userId: {}", userId);
    return awsS3Service.uploadFile(userId, category, multipartFiles);
  }
```
- `@RequestPart` 의 파라미터를 `List<MultipartFile>`로 받습니다.

### Service
```java
  public FileUploadResponse uploadFile(long userId, String category, List<MultipartFile> multipartFiles) {
    List<String> fileUrls = new ArrayList<>();

    // 파일 업로드 갯수를 정합니다(10개 이하로 정의)
    for (MultipartFile multipartFile : multipartFiles) {
      if (fileUrls.size() > 10) {
        throw new FileCountExceedException();
      }

      String fileName = PlandPMSUtils.buildFileName(userId, category, multipartFile.getOriginalFilename());
      ObjectMetadata objectMetadata = new ObjectMetadata();
      objectMetadata.setContentType(multipartFile.getContentType());

      try (InputStream inputStream = multipartFile.getInputStream()) {
        amazonS3Client.putObject(new PutObjectRequest(bucketName, fileName, inputStream, objectMetadata)
            .withCannedAcl(CannedAccessControlList.PublicRead));
        fileUrls.add(FILE_URL_PROTOCOL + bucketName + "/" + fileName);
      } catch (IOException e) {
        throw new FileUploadFailedException(e);
      }
    }

    return new FileUploadResponse(fileUrls);
  }
```
- 파일 갯수를 제외하곤 나머지는 동일합니다.


## 출처

- Spring-cloud-aws

[awspring/spring-cloud-aws](https://github.com/awspring/spring-cloud-aws/tree/v2.3.1)

- Spring S3 업로드

[Spring Cloud AWS S3 연동 및 파일 업로드](https://willseungh0.tistory.com/78?category=874439)

- Spring S3 다운로드

[AWS S3에서 파일 다운로드 받기](https://honinbo-world.tistory.com/81)

[OKKY | Content-Disposition의 한글 filename이 깨지는 이유? (추가. 인코딩된 파일이름 -> 한글 파일이름)](https://okky.kr/article/845566)

[How to return downloaded file name in Cyrillic?](https://stackoverflow.com/questions/53961134/how-to-return-downloaded-file-name-in-cyrillic)

- Spring S3 업로드 & 다운로드 유튜브

[Spring Boot With Amazon S3 : File Upload & Download Example | S3 Bucket | JavaTechie](https://www.youtube.com/watch?v=vY7c7k8xmKE)