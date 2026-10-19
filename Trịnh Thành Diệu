import SwiftUI
class Student: Identifiable {
    var id: String
    var name: String
    var age: Int

    init(id: String, name: String, age: Int) {
        self.id = id
        self.name = name
        self.age = age
    }

    func displayInfo() -> String {
        "ID: \(id) - Name: \(name) - Age: \(age)"
    }
}
class Subject: Identifiable {
    var id: String
    var name: String
    var credits: Int

    init(id: String, name: String, credits: Int) {
        self.id = id
        self.name = name
        self.credits = credits
    }
}

class StudentRecord: ObservableObject, Identifiable {
    let id = UUID()
    var student: Student
    var subject: Subject
    var year: Int

    @Published var mark: Double {
        didSet {
            if mark < 0 {
                mark = 0
            } else if mark > 10 {
                mark = 10
            }
        }
    }

    init(student: Student, subject: Subject, mark: Double, year: Int) {
        self.student = student
        self.subject = subject
        self.year = year
        self.mark = min(max(mark, 0), 10)
    }

    func setMark(_ mark: Double) {
        self.mark = min(max(mark, 0), 10)
    }

    func isPassed() -> Bool {
        mark >= 5.0
    }
}

class StudentData: ObservableObject {
    @Published var students = [
        Student(id: "S01", name: "A", age: 21),
        Student(id: "S02", name: "B", age: 21),
        Student(id: "S03", name: "C", age: 21),
        Student(id: "S04", name: "D", age: 21)
    ]

    @Published var subjects = [
        Subject(id: "M01", name: "iOS", credits: 3),
        Subject(id: "M02", name: "Math", credits: 4),
        Subject(id: "M03", name: "Physic", credits: 2),
        Subject(id: "M04", name: "English", credits: 2)
    ]

    @Published var records: [StudentRecord] = []

    init() {
        let sample: [(Int, Int, Double)] = [
            (0, 0, 8.5), (0, 1, 7.5), (1, 0, 6.0), (1, 2, 4.0), (2, 1, 9.0), (2, 3, 8.0), (3, 2, 5.5), (3, 3, 6.5)
        ]

        records = sample.map {
            StudentRecord(
                student: students[$0.0],
                subject: subjects[$0.1],
                mark: $0.2,
                year: 2026
            )
        }
    }

    func average(for student: Student) -> Double {
        let list = records.filter { $0.student.id == student.id }
        let credits = list.reduce(0) { $0 + $1.subject.credits }
        guard credits > 0 else { return 0 }

        let total = list.reduce(0.0) {
            $0 + $1.mark * Double($1.subject.credits)
        }

        return total / Double(credits)
    }

    func ranking(for student: Student) -> String {
        let avg = average(for: student)

        if avg >= 8.5 { return "Excellent" }
        if avg >= 7.0 { return "Good" }
        if avg >= 5.0 { return "Average" }
        return "Fail"
    }

    func register(
        student: Student,
        subject: Subject,
        mark: Double,
        year: Int
    ) -> Bool {
        let exists = records.contains {
            $0.student.id == student.id &&
            $0.subject.id == subject.id &&
            $0.year == year
        }

        if exists {
            return false
        }

        records.append(
            StudentRecord(
                student: student,
                subject: subject,
                mark: mark,
                year: year
            )
        )

        return true
    }

    func refreshResults() {
        objectWillChange.send()
    }
}

struct ContentView: View {
    @StateObject private var data = StudentData()
    @State private var refreshedAt = Date()

    var body: some View {
        NavigationStack {
            List {
                Section("Students") {
                    ForEach(data.students) { student in
                        VStack(alignment: .leading, spacing: 8) {
                            Text(student.name).font(.headline)
                            Text(student.displayInfo()).font(.caption)
                            Text("Average: \(data.average(for: student), specifier: "%.2f")")
                            Text("Ranking: \(data.ranking(for: student))").bold()

                            Text("Registered subjects").font(.subheadline)

                            ForEach(data.records.filter {
                                $0.student.id == student.id
                            }) {
                                RecordRow(record: $0)
                            }
                        }
                        .padding(.vertical, 6)
                    }
                }

                Section("Actions") {
                    Button("Refresh results") {
                        data.refreshResults()
                        refreshedAt = Date()
                    }

                    Text("Last refreshed: \(refreshedAt.formatted(date: .omitted, time: .standard))")
                        .font(.caption)
                }
            }
            .navigationTitle("Student Manager")
        }
    }
}

struct RecordRow: View {
    @ObservedObject var record: StudentRecord

    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text(record.subject.name)

                Text(record.isPassed() ? "Passed" : "Failed")
                    .font(.caption)
                    .foregroundStyle(
                        record.isPassed() ? .green : .red
                    )
            }
            Spacer()
            Text("Mark:")

            TextField(
                "Mark",
                value: Binding(
                    get: { record.mark },
                    set: { record.setMark($0) }
                ),
                format: .number.precision(.fractionLength(1))
            )
            .keyboardType(.decimalPad)
            .multilineTextAlignment(.trailing)
            .frame(width: 65)
            .textFieldStyle(.roundedBorder)
        }
    }
}
#Preview {
    ContentView()
}
