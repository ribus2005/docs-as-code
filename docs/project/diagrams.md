# Диаграммы

## Диаграмма последовательности

`````mermaid
sequenceDiagram

	participant input as Видеопоток

	participant system as Сервис

	participant YOLO as CNN-Детектор

	participant EffNet as CNN-Классификатор

	actor User as Диспетчер

	input->>system: Трансляция с устья

	system->>YOLO: Запуск детекции на кадре

	YOLO->>EffNet: Изображения людей в кадре

	EffNet-->>system: Вероятности ношения формы

	system->>User: Наличие людей без формы на устье
`````


## Диаграмма классов

```mermaid
classDiagram
    class VideoSource {
        +String id
        +String wellhead
        +String streamUrl
        +getFrame() Frame
    }

    class MonitoringService {
        +processFrame(sourceId, frame)
        +registerViolation(sourceId, detectedAt) Violation
    }

    class PersonDetector {
        +detect(frame) PersonDetection[]
    }

    class PersonDetection {
        +String boundingBox
        +float confidence
    }

    class UniformClassifier {
        +classify(frame, detection) UniformAssessment
    }

    class UniformAssessment {
        +float uniformProbability
        +bool hasUniform
    }

    class Violation {
        +String id
        +String sourceId
        +datetime detectedAt
        +bool confirmed
        +String confirmedBy
        +confirm(dispatcherId)
    }

    class ViolationRepository {
        +save(violation)
        +findByPeriod(start, end) Violation[]
    }

    class NotificationService {
        +notify(violation, dispatcher)
    }

    class Dispatcher {
        +String id
        +String notificationAddress
        +confirmViolation(violationId)
    }

    class ReportService {
        +buildSummary(start, end) ViolationSummary
    }

    class ViolationSummary {
        +datetime periodStart
        +datetime periodEnd
        +int totalCount
        +Map countBySource
    }

    class Customer {
        +String organization
        +requestSummary(start, end) ViolationSummary
    }

    MonitoringService --> VideoSource : получает кадры
    MonitoringService --> PersonDetector : обнаруживает людей
    PersonDetector ..> PersonDetection : создаёт
    MonitoringService --> UniformClassifier : проверяет форму
    UniformClassifier ..> PersonDetection : анализирует
    UniformClassifier ..> UniformAssessment : возвращает
    MonitoringService ..> Violation : регистрирует
    VideoSource "1" <-- "0..*" Violation : источник
    MonitoringService --> ViolationRepository : сохраняет события
    ViolationRepository --> Violation : хранит
    MonitoringService --> NotificationService : инициирует уведомление
    NotificationService ..> Violation : включает время и источник
    NotificationService --> Dispatcher : уведомляет
    Dispatcher ..> Violation : подтверждает
    Dispatcher --> ViolationRepository : сохраняет подтверждение
    Customer --> ReportService : запрашивает сводку
    ReportService --> ViolationRepository : получает события за период
    ReportService ..> ViolationSummary : формирует
```
