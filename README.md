from pathlib import Path
import hashlib


class DuplicateImageFinder:
    IMAGE_EXTENSIONS = {
        ".jpg", ".jpeg", ".png",
        ".gif", ".bmp", ".webp"
    }

    def __init__(self, directory):
        self.directory = Path(directory)
        self.hashes = {}
        self.duplicates = []

    def file_hash(self, file_path):
        hasher = hashlib.sha256()

        with open(file_path, "rb") as f:
            while chunk := f.read(8192):
                hasher.update(chunk)

        return hasher.hexdigest()

    def scan(self):

        for file in self.directory.rglob("*"):

            if (
                file.is_file()
                and file.suffix.lower()
                in self.IMAGE_EXTENSIONS
            ):

                try:
                    file_hash = self.file_hash(file)

                    if file_hash in self.hashes:

                        self.duplicates.append(
                            (
                                self.hashes[file_hash],
                                file
                            )
                        )

                    else:
                        self.hashes[file_hash] = file

                except Exception as e:
                    print(
                        f"Error: {file} -> {e}"
                    )

    def report(self):

        if not self.duplicates:
            print(
                "\nNo duplicate images found."
            )
            return

        print(
            f"\nFound "
            f"{len(self.duplicates)} "
            f"duplicate images:\n"
        )

        for original, duplicate in self.duplicates:

            print("=" * 70)
            print(f"Original : {original}")
            print(f"Duplicate: {duplicate}")


def main():

    folder = input(
        "Enter image folder path: "
    )

    finder = DuplicateImageFinder(folder)

    print("\nScanning...\n")

    finder.scan()

    finder.report()


if __name__ == "__main__":
    main()
