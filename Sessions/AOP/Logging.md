  #todo
@Audited(targetAuditMode = NOT_AUDITED)  
@EnableJpaAuditing(auditorAwareRef = "AuditorAwareImpl")  
@EntityListeners(AuditingEntityListener.class)  
@Slf4j

base enitity