// Role.cs
public class Role
{
    public int RoleId { get; set; }
    public string RoleName { get; set; }
}

// User.cs
public class User
{
    public int UserId { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
    public string PasswordHash { get; set; }
    public int RoleId { get; set; }
    public DateTime CreatedAt { get; set; }
    public Role Role { get; set; }
}

// Course.cs
public class Course
{
    public int CourseId { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public List<Module> Modules { get; set; }
}

// Module.cs
public class Module
{
    public int ModuleId { get; set; }
    public int CourseId { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public int OrderIndex { get; set; }
    public DateTime CreatedAt { get; set; }
    public Course Course { get; set; }
    public List<Lesson> Lessons { get; set; }
}

// Lesson.cs
public class Lesson
{
    public int LessonId { get; set; }
    public int ModuleId { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public int OrderIndex { get; set; }
    public DateTime CreatedAt { get; set; }
    public Module Module { get; set; }
    public List<Content> Contents { get; set; }
    public List<Question> Questions { get; set; }
}

// Content.cs
public class Content
{
    public int ContentId { get; set; }
    public int LessonId { get; set; }
    public string Title { get; set; }
    public string ContentText { get; set; }
    public string FilePath { get; set; }
    public string ContentType { get; set; }
    public int OrderIndex { get; set; }
    public DateTime CreatedAt { get; set; }
    public Lesson Lesson { get; set; }
}

// Question.cs
public class Question
{
    public int QuestionId { get; set; }
    public int LessonId { get; set; }
    public string QuestionText { get; set; }
    public string QuestionType { get; set; }
    public DateTime CreatedAt { get; set; }
    public Lesson Lesson { get; set; }
    public List<Answer> Answers { get; set; }
}

// Answer.cs
public class Answer
{
    public int AnswerId { get; set; }
    public int QuestionId { get; set; }
    public string AnswerText { get; set; }
    public bool IsCorrect { get; set; }
    public Question Question { get; set; }
}

// UserProgress.cs
public class UserProgress
{
    public int ProgressId { get; set; }
    public int UserId { get; set; }
    public int LessonId { get; set; }
    public bool IsCompleted { get; set; }
    public int TimeSpent { get; set; }
    public DateTime? LastVisited { get; set; }
    public User User { get; set; }
    public Lesson Lesson { get; set; }
}

// QuizResult.cs
public class QuizResult
{
    public int ResultId { get; set; }
    public int UserId { get; set; }
    public int LessonId { get; set; }
    public int Score { get; set; }
    public int TotalQuestions { get; set; }
    public DateTime StartedAt { get; set; }
    public DateTime? CompletedAt { get; set; }
    public User User { get; set; }
    public Lesson Lesson { get; set; }
}
