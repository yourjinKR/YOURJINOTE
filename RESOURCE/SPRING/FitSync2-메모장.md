
```java
package com.fitsync.domain.attachment.entity;

import com.fitsync.domain.user.entity.User;
import jakarta.persistence.*;
import lombok.*;
import org.hibernate.annotations.Comment;
import org.hibernate.annotations.CreationTimestamp;

import java.time.OffsetDateTime;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
@EqualsAndHashCode(of = "id")
@Entity
@Table(
        name = "attachments",
        indexes = {
                @Index(name = "idx_attachments_uploader", columnList = "uploader_user_id"),
                @Index(name = "idx_attachments_attachable_order", columnList = "attachable_type, attachable_id, display_order")
        }
)
@Comment("프로필, 채팅 등 모든 첨부파일을 관리하는 중앙 테이블")
public class Attachment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // 1) 파일 자체의 고유 정보
    @Column(name = "original_filename", nullable = false, length = 255)
    @Comment("사용자가 업로드한 원본 파일명")
    private String originalFilename;

    @Column(name = "file_url", nullable = false, columnDefinition = "TEXT")
    @Comment("파일 저장소(S3 등)의 접근 URL")
    private String fileUrl;

    @Column(name = "storage_public_id", unique = true, length = 500)
    @Comment("저장소에서 파일을 식별하는 고유 키 (예: S3 Object Key)")
    private String storagePublicId;

    @Column(name = "file_size_bytes")
    @Comment("파일 크기(바이트)")
    private Long fileSizeBytes;

    @Column(name = "mime_type", length = 100)
    @Comment("파일 MIME 타입 (예: image/jpeg)")
    private String mimeType;

    @Column(name = "file_extension", length = 10)
    @Comment("파일 확장자 (예: jpg, png)")
    private String fileExtension;

    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false, columnDefinition = "TIMESTAMPTZ")
    private OffsetDateTime createdAt;

    // 2) 업로더 (N:1, nullable) - DB 레벨에서 ON DELETE SET NULL (JPA는 DB 제약에 위임)
    @ManyToOne(fetch = FetchType.LAZY, optional = true)
    @JoinColumn(name = "uploader_user_id",
            foreignKey = @ForeignKey(name = "fk_uploader_user"))
    @Comment("파일을 업로드한 회원 (users.id)")
    private User uploader;

    // 3) 다형적 연결 지점 (type + id)
    @Column(name = "attachable_type", nullable = false, length = 50)
    @Enumerated(EnumType.STRING)
    @Comment("파일이 첨부된 도메인의 타입 (예: USER_PROFILE, POST, CHAT_MESSAGE 등)")
    private AttachType attachableType;

    @Column(name = "attachable_id", nullable = false)
    @Comment("첨부 대상 도메인의 실제 ID")
    private Long attachableId;

    // 4) 정렬
    @Column(name = "display_order", nullable = false)
    @Comment("목록 내 정렬 순서 (작을수록 우선)")
    private Short displayOrder;

    /* ==== 편의 메서드 ==== */

    public void changeDisplayOrder(short newOrder) {
        this.displayOrder = newOrder;
    }

    /**
     * 다형적 연결 변경 (전환/이동)
     */
    public void rebindTo(AttachType newType, Long newId) {
        this.attachableType = newType;
        this.attachableId = newId;
    }

    /**
     * 업로더 변경(또는 분리)
     */
    public void changeUploader(User newUploader) {
        this.uploader = newUploader; // null 허용
    }

    /**
     * 메타 정보 갱신 (URL/용량/MIME/확장자 등)
     */
    public void updateFileMeta(String fileUrl, String storagePublicId,
                               Long fileSizeBytes, String mimeType, String fileExtension) {
        this.fileUrl = fileUrl;
        this.storagePublicId = storagePublicId;
        this.fileSizeBytes = fileSizeBytes;
        this.mimeType = mimeType;
        this.fileExtension = fileExtension;
    }

    /**
     * 원본 파일명 교체
     */
    public void renameOriginal(String newOriginalFilename) {
        this.originalFilename = newOriginalFilename;
    }
}

```


```java
package com.fitsync.domain.attachment.entity;

public enum AttachType {
    USER_PROFILE, EXERCISE_INFO, POST, CHAT
}

```


```java

```